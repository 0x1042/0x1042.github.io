# formatter 

<ins>**为自定义类型实现formater的方式是特化`std::formatter<Type,CharT>`**</ins>

- 实现`parse` 函数, 解析格式字符串中的说明符(即冒号 : 后面的部分) 
- 实现`format` 函数, 执行实际的格式化输出

## 基础`struct` 

```C++
struct Tag {
    std::string key;
    std::string val;
};

template <>
struct std::formatter<Tag> {
    // 1. parse: 解析格式字符串
    // 上下文 ctx 包含了格式字符串的迭代器
    static constexpr auto parse(std::format_parse_context& ctx) { return ctx.begin(); }

    // 2. format: 输出内容
    // const Point& p: 要格式化的对象
    // FormatContext& ctx: 格式化上下文，提供输出迭代器
    static auto format(const Tag& tag, std::format_context& ctx) {
        // std::format_to 向 ctx.out() 写入数据
        return std::format_to(ctx.out(), "({} -> {})", tag.key, tag.val);
    }
};
```

## 容器 

```C++
template <typename V>
struct RangeFormatter {
    constexpr auto parse(std::format_parse_context& ctx) { return ctx.begin(); }

    static auto format(const V& range, std::format_context& ctx) {
        auto out = ctx.out();
        out = std::format_to(out, "[");
        bool first = true;
        for (const auto& tmp : range) {
            if (!first) {
                out = std::format_to(out, ", ");
            }
            first = false;
            // 递归格式化元素：item 会自动寻找它自己的 formatter
            out = std::format_to(out, "{}", tmp);
        }
        return std::format_to(out, "]");
    }
};

template <>
struct std::formatter<std::vector<Tag>> : RangeFormatter<std::vector<Tag>> {};
```

## 嵌套

```C++
struct Item {
    int64_t id = 0;
    std::string brand;
    double coef = 1.0;
    std::vector<Tag> tags;
};

template <>
struct std::formatter<Item> {
    static constexpr auto parse(std::format_parse_context& ctx) { return ctx.begin(); }

    static auto format(const Item& item, std::format_context& ctx) {
        auto out = ctx.out();
        out = std::format_to(out, "id:{} brand:{} coef:{} ", item.id, item.brand, item.coef);
        return std::format_to(out, "tags:{}", item.tags);
    }
};
```

## log

- `std::make_format_args`: 将变参模板参数包转换为类型擦除的参数包。
- `std::format_args`: 存储参数包的类型，用于传递给格式化函数。
- `std::vformat`: 接收格式字符串和 format_args，生成最终的字符串

```C++
template <typename... Args>
void log_impl(const std::string& fmt, Args&&... args) {
    auto fmt_args = std::make_format_args(args...);
    std::string out = std::vformat(fmt, fmt_args);
    std::cout << "[LOG] " << out << '\n';
}
```

```C++
enum class LogLevel : uint8_t { INFO, WARN, ERROR };

void log_impl(LogLevel level, const std::source_location& loc, std::string_view message) {
    // 获取当前时间
    auto now = std::chrono::system_clock::now();

    // 简单的颜色代码
    const char* color_code = "";
    const char* level_str = "";

    switch (level) {
        case LogLevel::INFO:
            level_str = "INFO";
            color_code = "\033[32m";
            break; // Green
        case LogLevel::WARN:
            level_str = "WARN";
            color_code = "\033[33m";
            break; // Yellow
        case LogLevel::ERROR:
            level_str = "ERROR";
            color_code = "\033[31m";
            break; // Red
    }
    const char* reset_code = "\033[0m";

    // 格式化最终日志条目
    // C++20 chrono 也可以直接被 format
    // {:%F %T} 是 YYYY-MM-DD HH:MM:SS 格式
    std::string final_log = std::format(
        "[{:%F %T}] [{}]{} [{}:{}] {}\n",
        std::chrono::floor<std::chrono::seconds>(now), // 时间
        level_str, // 等级文本
        color_code, // 颜色开始
        loc.file_name(), // 文件名
        loc.line(), // 行号
        message // 用户格式化好的消息
    );

    // 输出并重置颜色
    std::cout << final_log << reset_code;
    // 注意：如果是 ERROR 可以输出到 std::cerr
}

template <typename... Args>
void log(
    LogLevel level,
    const std::source_location& loc, // 需要显式传递或通过宏封装
    std::format_string<Args...> fmt, // 编译期检查格式串
    Args&&... args) {
    // 核心步骤：将用户参数格式化为一条纯文本消息
    std::string user_msg = std::format(fmt, std::forward<Args>(args)...);

    // 调用实现层
    log_impl(level, loc, user_msg);
}

#define LOG_INFO(fmt, ...) log(LogLevel::INFO, std::source_location::current(), fmt, __VA_ARGS__)
#define LOG_WARN(fmt, ...) log(LogLevel::WARN, std::source_location::current(), fmt, __VA_ARGS__)
#define LOG_ERROR(fmt, ...) log(LogLevel::ERROR, std::source_location::current(), fmt, __VA_ARGS__)
```
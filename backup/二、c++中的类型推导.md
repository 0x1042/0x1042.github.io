- [类型推导](#类型推导)
- [`auto`](#auto)
- [`decltype`](#decltype)
  - [不带括号(获取的是标识符 定义时的类型)](#不带括号获取的是标识符-定义时的类型)
  - [带括号 获取表达式的值类别](#带括号-获取表达式的值类别)
- [`decltype(auto)`](#decltypeauto)
- [`CTAD` 类模板参数推导](#ctad-类模板参数推导)

# 类型推导 

# `auto`

**auto 是值语义，即通过移动/拷贝构造，不会保留cv属性，如果需要保留cv属性，需要显式指定**

```cpp
class Cat {};

auto get_cat() -> Cat *;

auto get_const_cat() -> const Cat *;


Cat cat{};
Cat * cat1 = &cat;
const Cat * cat2 = &cat;

Cat & lr_cat = cat;
const Cat & ltc_cat = cat;
Cat&& rr_cat = Cat{};
```

| 表达式                         | auto推导的类型 |
| ------------------------------ | -------------- |
| `auto ccat1 = cat`             | `Cat`          |
| `auto ccat2 = cat1`            | `Cat*`         |
| `auto ccat3 = cat2`            | `const Cat*`   |
| `auto ccat4 = get_cat()`       | `Cat*`         |
| `auto ccat5 = get_const_cat()` | `const Cat*`   |
| `auto ccat6 = lr_cat`          | `Cat`          |
| `auto ccat7 = ltc_cat`         | `Cat`          |
| `auto ccat8 = rr_cat`          | `Cat`          |
| `auto & ccat9 = lr_cat`        | `Cat&`         |
| `const auto & ccat10 = lr_cat` | `const Cat&`   |
| `auto & ccat11 = ltc_cat`      | `const Cat&`   |
| `auto && ccat12 = cat`         | `Cat&`         |
| `auto && ccat13 = Cat{}`       | `Cat&&`        |


# `decltype`

> 作用：获取 标识符被定义时的类型或者 整体作为 表达式 时的值类别

- 参数带括号 decltype((T))，获取作为表达式时的 值类别
- 参数不带括号 decltype(T), 获取标识符 定义时的类型 

```cpp
class Student {
public:
    uint32_t id{0};
    std::string name;
};

inline void test_decltype() {
    Student student;
    Student * st_ptr = &student;
    const Student * st_cptr = &student;
    Student & st_ref = student;
    Student && st_tmp = {};
}
```

## 不带括号(获取的是标识符 定义时的类型)

| 表达式                                            | 类型             |
| ------------------------------------------------- | ---------------- |
| `using T1 = decltype(student)`                    | `Student`        |
| `using T2 = decltype(st_ptr)`                     | `Student*`       |
| `using T3 = decltype(st_cptr)`                    | `const Student*` |
| `using T4 = decltype(st_ref)`                     | `Student&`       |
| `using T5 = decltype(st_tmp)`                     | `Student&&`      |
| `using T6 = decltype(student.id)`                 | `uint32_t`       |
| `using T7 = decltype(Student{1024, "张三"}.name)` | `std::string`    |

## 带括号 获取表达式的值类别

| 表达式                                              | 类型               |
| --------------------------------------------------- | ------------------ |
| `using T1 = decltype((student))`                    | `Student&`         |
| `using T2 = decltype((st_ptr))`                     | `Student* &`       |
| `using T3 = decltype((st_cptr))`                    | `const Student* &` |
| `using T4 = decltype((st_ref))`                     | `Student&`         |
| `using T5 = decltype((st_tmp))`                     | `Student&`         |
| `using T6 = decltype((student.id))`                 | `uint32_t&`        |
| `using T7 = decltype((Student{1024, "张三"}))`      | `Student`          |
| `using T8 = decltype((Student{1024, "张三"}.name))` | `std::string&&`    |
| `using T9 = decltype((++student.id))`               | `uint32_t&`        |
| `using T10 = decltype((student.id++))`              | `uint32_t`         |

- 如果表达式是左值，那么 `decltype((exp))` 就是左值引用（T1->T6）
- `st_tmp`的类型是 右值引用，但是作为表达式，可以被取地址，所以是左值引用
- `T7` 原始表达式是一个纯右值，`decltype((exp))` 是右值（不带引用）
- `T8` 是一个将亡值
- `T9` ++x作为表达式是左值
- `T10` x++作为表达式是右值

# `decltype(auto)`

> 默认使用`auto`时，丢失了引用性和`cv`属性，若指明了 `const`属性，则导致结果始终为`const`，若采用引用，则需要显示指定`auto&` 或者 `auto&&`,这又会导致只能表现为 引用语义。`decltype(auto)` 用于精确获取等号右边类型的场景.

```cpp

// Student s1;
decltype(student) s1 = {1, "李四"};
// uint32_t s3;
decltype(student.id) s3 = 1024;

// 无需再声明每一种类型
decltype(auto) name1 = "王五";
decltype(auto) name2 = std::string("王五");

// stud_type 是Student&
decltype(auto) stud_type = (student);
```
- `std::forward` 用于精确保留参数的类型
- `decltype(auto)` 可以精确获取函数的返回值

**如何获取任意的函数以及给定的参数对应的返回值 ?**

- 使用标准库的 `std::invoke_result`
- 使用`std::declval`元函数 

```cpp
template <typename F, typename... Args>
using CallResult = decltype(std::declval<F>()(std::declval<Args>()...));

struct Callable {
    auto operator()(double x, double y) -> double { return x + y; }
    auto operator()(const std::string & x, const std::string & y) -> std::string { return x + y; }
};

inline void test_result2() {
    using T1 = CallResult<Callable, double, double>;
    using T2 = CallResult<Callable, std::string, std::string>;



    std::cerr << "typeof t1:" << typeid(T1).name() << '\n';
    std::cerr << "typeof t1:" << typeid(T2).name() << '\n';

    using T3 = std::invoke_result_t<Callable, double, double>;
    using T4 = std::invoke_result_t<Callable, std::string, std::string>;

    std::cerr << "typeof t3:" << typeid(T3).name() << '\n';
    std::cerr << "typeof t4:" << typeid(T4).name() << '\n';
}
```

**`std::declval()` 只能在`decltype`或`sizeof`等非求值上下文中，当类模版参数不可构造或者构造函数是私有的时，可以使用std::declval<F>() 构造实例，等同`using CallResult = decltype(F{}(Args{}...));`**


# `CTAD` 类模板参数推导

> 全称是 Class Template Args Deduction, 编译器通过模板参数推导其构造函数的模板参数类型，简化类型定义 

```cpp

// p1 与 p2 等价，但是p1 的定义更简单
auto p1 = std::make_pair(1, 3.14);
auto p2 = std::make_pair<int, double>(1, 3.14);
```

**自定义推导规则**

- 考虑一下代码 
    ```cpp
    template <typename F, typename S> 
    class MyPair {
    public:
        F first;
        S second;

        MyPair(const F & first, const S & second) : first(first), second(second) {}
    };
    ```

- 测试
    ```cpp
    // success
    MyPair p1(3.14, 3.14);
    MyPair p2(3.14, 2);

    // fail
    MyPair p3(3.14, "hello world");
    ```
- 解决办法 
    ```cpp
    // 自定义类型推导
    // 告诉编译器，对于MyPair(X,Y)方式构造程序时，自行推导出 模板类 MyPair<X,Y> 
    template <typename F, typename S>
    MyPair(F, S) -> MyPair<F, S>;
    ```
# concept

- [concept](#concept)
  - [基本定义和使用](#基本定义和使用)
  - [约束表达式](#约束表达式)
  - [`requires` 表达式](#requires-表达式)
  - [`requires` 子句](#requires-子句)
  - [`concepts` header](#concepts-header)

## 基本定义和使用

- 基本定义
```cpp
// 语法格式
// template<typename T>
// concept concept_name = constraint-expression(约束表达式);

template <typename T>
concept integral = std::is_integral_v<T>;

template <typename T>
concept floating_point = std::is_floating_point_v<T>;

template <typename T>
concept C = std::is_integral_v<T> || (sizeof(T) > 1);

template <typename T, typename U>
concept Derived = std::is_base_of_v<U, T>;

```

- 基本使用 **使用`concept_name` 替换 `typename`**

```cpp

// 定义 
class Base {
public:
    [[nodiscard]] auto getValue() const -> int32_t { return value; }

    explicit Base(const int32_t value)
        : value(value) {
    }

private:
    int32_t value;
};

template <typename T>
concept DerivedBase = std::is_base_of_v<Base, T>;

// 使用concept_name 替换 typename
template <DerivedBase T>
auto doGetValue(const T & t) -> int32_t {
    return t.getValue();
}

class DerivedClass : public Base {
public:
    explicit DerivedClass(int32_t value)
        : Base(value) {
    }

    [[nodiscard]] auto getValue() const -> int32_t { return 1024; }
};
```

## 约束表达式

- 合取式 conjunctions，逻辑与 
```cpp
template <typename T>
concept integral = std::is_integral_v<T>;

template <typename T>
concept signed_int = integral<T> && std::is_signed_v<T>;

```
- 析取式 disjunctions，逻辑或

```cpp
template <typename T>
concept integral = std::is_integral_v<T>;

template <typename T>
concept floating_point = std::is_floating_point_v<T>;

template <typename T>
concept number = integral<T> || floating_point<T>;

```
- 原子约束 atomic constraints 

```cpp
```

## `requires` 表达式

> 除了使用`type traits` 定义概念之外，也可以使用 `requires` 表达式来表达对模板参数及其对象的特征要求
> 成员函数、自由函数、关联类型等

- 基本语法 

```cpp

requires(可选参数列表) {
    表达式1
    表达式2
}

```

- 基本约束 
```cpp
template <typename T>
concept Machine = requires(T t)
{
    // 要求存在同名的成员函数
    t.powerup();

    t.powerDown();
    // 要求存在成员变量name
    t.name;
    // 要求存在静态成员count
    T::count;
};

template <typename T>
concept Animal = requires(T t1, T t2, T t3)
{
    // 要求存在 name 成员变量
    t1.name;

    // 要求能够进行判等操作
    t1 == t2;

    // 要求能够进行 加、乘操作
    t1 + t2 * t3;
};

```

- 类型约束 
```cpp
template <typename T>
concept C2 = requires
{
    // 要求存在类型成员 type
    typename T::type;

    // 要求能够与vector 组合，实现模板实例化
    typename std::vector<T>;
};
```

- 组合约束 **需要大括号括起来**
```cpp
template <typename T>
concept C3 = requires(T t1, T t2) {
    // 表达式不能有异常
    { t1 = std::move(t2) } noexcept;

    // 要求接口返回类型与T一致
    { t1.get_info() } -> std::same_as<T>;

    // 要求接口返回类型能够转换成float
    { t1.get_unit() } -> std::convertible_to<float>;
};

```

- 嵌套约束 
```cpp
template <typename T>
concept C3 = requires {
    requires sizeof(T) > 4;
};
```

## `requires` 子句

> 用于判断所约束的类型在上下文中 是否可行

**上下文** 感觉是废话，concept 不就是为了简化模板编程么，肯定只能在模板编程中存在 

1. 函数模板
2. 模板类 
3. 模板类的成员函数 

```cpp
// 这里是 require 子句(区别与require 表达式)
template <typename T>
    requires std::is_integral_v<T>
auto add(T t1, T t2) -> T {
    return t1 + t2;
}

// 编译成功
add(1, 2);
//  note: candidate template ignored: constraints not satisfied [with T = double]
//  note: because 'std::is_integral_v<double>' evaluated to false
// add(1.2, 2.2);
```

## `concepts` header

```cpp

// 想同类
template <typename T, typename U>
concept same_as = std::is_same_v<T, U>;

// 是否是派生关系 
template <typename Base, typename Derived>
concept derived_from = std::is_base_of_v<Base, Derived> && std::is_convertible_v<std::add_cv_t<Derived *>, std::add_cv_t<Base *>>;

// 是否可转换
template <typename F, typename T>
concept convertible_to = std::is_convertible_v<F, T> && requires(std::add_rvalue_reference_t<F> (&f)()) { static_cast<T>(f()); };

```
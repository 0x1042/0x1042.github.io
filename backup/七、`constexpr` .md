# constexpr 元编程

- [constexpr 元编程](#constexpr-元编程)
  - [`constexpr` 变量](#constexpr-变量)
  - [`constinit` 初始化](#constinit-初始化)
  - [`constexpr` 函数](#constexpr-函数)
  - [`consteval` 函数](#consteval-函数)
- [`if constexpr`](#if-constexpr)
- [折叠表达式](#折叠表达式)
  - [右折叠](#右折叠)
  - [左折叠](#左折叠)
- [令人头大的`const`](#令人头大的const)
  - [`int const* ptr`](#int-const-ptr)
  - [`int * const ptr`](#int--const-ptr)
  - [终极CASE](#终极case)

## `constexpr` 变量

**与`const`变量的区别？**

- `constexpr` 需要保证表达式可在编译时求值，否则会出现编译错误 
- `const` 表达式拥有常量性。若能咋编译时求值，也可以用于编译时计算上下文，否则将用于运行时求值
  - 数组下标 
  - 模板参数 
  - 静态断言
  - 初始化`constexpr`常量等

- `const`只能用于非静态成员的函数而不是所有函数,它保证成员函数不修改任何非静态数据。

```cpp
constexpr double PI = 3.141592653;

template <char c>
constexpr bool is_digit = (c >= '0' && c <= '9');


template <size_t N>
constexpr size_t fib = fib<N - 1> + fib<N - 2>;

template <>
constexpr size_t fib<0> = 0;

template <>
constexpr size_t fib<1> = 1;

TEST_CASE("test digit", "[is_digit]") {
    LOG(INFO) << "is_digit:c:" << local4::is_digit<'c'>;
    LOG(INFO) << "is_digit:7:" << local4::is_digit<'7'>;
}
```

## `constinit` 初始化

> constinit的作用在于显式地指定变量的初始化方式为静态初始化。所有使用constinit声明的变量，其生命周期必须为静态生命周期或线程本地（Thread-local）生命周期（即不能为局部变量），其初始化表达式必须是一个常量表达式

`constexpr` 定义的变量被要求可以在编译时求值，它们需要拥有常量的属性。而`constinit` 定义的变量同样要求在编译时能对表达式求值，但是仍然保留了可变的属性.

`C++`中各个编译单元的全局变量在运行时的初始化顺序是不固定的，若这些全局变量存在依赖关系，初始化的结果是未定义的. 常规的解决方案是使用函数的局部静态变量来代替全局变量，将直接的依赖关系转换成函数间的依赖关系，从而使用函数的调用关系来确保局部静态变量的初始化顺序.

`constinit` 要求全局生命周期的变量能够在编译时初始化，既可以解决初始化顺序不固定的问题，同时也节约了运行时开销.

```cpp

consteval auto foo() -> double {
    return PI;
}

constinit int a = 10;
constinit double pi = foo();
constinit thread_local int c = 20;
```

## `constexpr` 函数

表示函数**可能**在编译时求值.同时，`constexpr` 函数修饰的函数意味着它是`inline`的

```cpp
template <typename T>
concept number = std::is_integral_v<T> || std::is_floating_point_v<T>;

template <number T>
constexpr auto min(std::initializer_list<T> nums) -> T {
    T low = std::numeric_limits<T>::max();

    for (const auto item : nums) {
        low = item < low ? item : low;
    }
    return low;
}
```

## `consteval` 函数

表示函数**必须**能在编译时求值，否则报错

```cpp
template <typename T>
concept number = std::is_integral_v<T> || std::is_floating_point_v<T>;

template <number T>
consteval auto min(std::initializer_list<T> nums) -> T {
    T low = std::numeric_limits<T>::max();

    for (const auto item : nums) {
        low = item < low ? item : low;
    }

    return low;
}

TEST_CASE("test min", "[min]") {
    LOG(INFO) << "min int value:" << min({1, 2, 3, 4, 5, 6});
    LOG(INFO) << "min float value:" << min({1.1, 2.2, 3.3, 4.4, 5.5, 6.6});
}
```

# `if constexpr`

> 可以替代之前的 `std::enable_if`

```cpp
template <class T>
constexpr auto absolute(T arg) -> T {
    return arg < 0 ? -arg : arg;
}

template <class T>
constexpr auto close_enough(T a, T b) -> std::enable_if_t<std::is_floating_point_v<T>, bool> {
    return absolute(a - b) < std::numeric_limits<T>::epsilon();
}

template <class T>
constexpr auto close_enough(T a, T b) -> std::enable_if_t<not std::is_floating_point_v<T>, bool> {
    return a == b;
}

template <typename T>
constexpr auto close_enough2(T a, T b) -> bool {
    if constexpr (std::is_floating_point_v<T>) {
        return std::fabs(a - b) < std::numeric_limits<T>::epsilon();
    }

    return a == b;
}
```

# 折叠表达式

> 用于简化一部分需要模板递归才能实现的循环功能 ，从语法上分为左折叠和右折叠 
> 语义是，对参数包中的值进行迭代，每次迭代将当前的值与累计值急性二元操作求值，求值的结果作为下一次累计值进行二元迭代

```cpp

// 右折叠 
(pack op ...[op init])

// 左折叠
([init op]... op pack)
```

- pack: 模板参数包 
- op： 二元操作符
- init: 可选的初始累计值

## 右折叠

```cpp

/**
 * @brief 右折叠
 * 
 * @tparam IS 参数包
 * + 二元操作符 
 * 0 初始值
 */
template <size_t... IS>
constexpr int rsum = (IS + ... + 0);

```

## 左折叠 

```cpp
/**
 * @brief 左折叠
 * 
 * @tparam IS 
 */
template <size_t... IS>
constexpr int lsum = (0 + ... + IS);
```


# 令人头大的`const`

> 以变量名开始顺时针旋转

- [https://github.com/Eisenwave/cdecl-plus](https://github.com/Eisenwave/cdecl-plus)
- [https://c-faq.com/decl/spiral.anderson.html](https://c-faq.com/decl/spiral.anderson.html)
- [what-is-the-difference-between-const-int-const-int-const-and-int-const](https://stackoverflow.com/questions/1143262/what-is-the-difference-between-const-int-const-int-const-and-int-const)

<img width="736" alt="const" src="https://github.com/0x1042/0x1042.github.io/assets/7525242/efdc995b-7391-4675-8c01-b66720f32021">


## `int const* ptr`

```cpp
int const* ptr;
```

- `ptr` 右时针旋转，首先遇到`*`，`ptr`是一个指针
-  然后，`const`右时针旋转，遇到`int`
- 所以，`ptr` 是一个指向 `const int` 的 指针
- 与`const int* ptr` 含义一样


## `int * const ptr`

```cpp
int * const ptr;
```

- `ptr` 右时针旋转，遇到`const`, `ptr`是一个`const` 变量
- `*` 右时针旋转，遇到`int`
- 所以，`ptr` 是一个 `const `指针，指向 `int` 变量


## 终极CASE

```cpp
void (*signal(int, void (*fp)(int)))(int);
```

1. `signal` 是一个函数，它接受两个参数，一个是整数 `(int)`，另一个是一个函数指针 `(void (*fp)(int))`
2. `int`, 表示第一个参数是一个整数
3. `void (*fp)(int)`, 表示第二个参数是一个函数指针，该指针指向一个函数，该函数接受一个整数参数并返回`void`
4. 整个声明最外层的返回类型是 void (*) (int)，表示 `signal` 函数返回一个函数指针
5. 这个函数指针指向一个函数，该函数接受一个整数参数并返回 `void`

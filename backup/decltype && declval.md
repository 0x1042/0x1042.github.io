
# decltype 

<ins>**decltype是一个关键字,而不是函数,用于在编译期推断一个表达式的类型,不会实际计算表达式的值**</ins>
<ins>**对于任何被声明的变量 x，decltype((x)) 永远返回一个左值引用**</ins>

> [!IMPORTANT]
> `decltype` 的参数是「变量名」或者「表达式」. 
> 当参数是 变量名时，返回变量声明时的类型 
> 当参数是 表达式时，返回表达式的类型和值类别


1. 如果表达式是一个无括号的标识符或类成员访问, 返回该实体的声明类型
2. 如果表达式是函数调用或函数调用形式的操作符重载,返回函数返回类型
3. 如果表达式是左值，返回T&
4. 如果表达式是将亡值（xvalue），返回 T&&
5.  如果表达式是纯右值（prvalue），返回 T

```C++
int32_t x = 10;
int32_t y = 20; 
decltype(x);  参数是x, 类型是「变量名」，返回 int32_t
decltype((x)); 参数是 (x), 类型是 「表达式」,

```


> [!NOTE]
> decltype(expression): 如果 expression 是一个未加括号的 id-expression (标识符表达式，如变量名) 或者一个未加括号的类成员访问表达式 ( object.member 或 pointer->member ), 返回该标识符或成员被声明时的类型 (declared type)
> - 例如，如果 int x = 0;，那么 decltype(x) 就是 int。
> - 如果 const int& y = x;，那么 decltype(y) 就是 const int&。
> - 如果 struct S { double m; }; S s;，那么 decltype(s.m) 就是 double

> [!NOTE]
> `decltype((expression))`: 如果 `expression` 是任何其他类型的表达式 (例如，函数调用、字面量、带括号的标识符、算术运算等), 返回表达式的值类别 (value category) 和类型.
> - 如果 `expression` 的结果是一个左值 (lvalue)，类型为 T，那么 `decltype(expression)` 是 T& (左值引用)
> - 如果 `expression` 的结果是一个将亡值 (xvalue)，类型为 T，那么 `decltype(expression)` 是 T&& (右值引用)
> - 如果 `expression` 的结果是一个纯右值 (prvalue)，类型为 T，那么 `decltype(expression)` 是 T
> - 如果 `expression` 是一个函数，那么 `decltype(expression)` 是函数的返回值类型
> - 重要： `const 和 volatile` 限定符会被保留。

```c++
int x = 0;
const int& y = x;
int func();

struct A { double d; };
A a;
```

| expression               | 结果                                        |
| ------------------------ | ------------------------------------------- |
| `decltype(x)`            | x是无括号的标识符，返回x的类型，int         |
| `decltype(a.d)`          | a.d是类成员变量访问，返回double             |
| `decltype(func())`       | 参数是函数调用，返回func的返回值，即int     |
| `decltype((x))`          | (x) 是左值，返回T&，即int&                  |
| `decltype(x+y)`          | x+y是纯右值，无法对&(x+y)取地址,返回T,即int |
| `decltype(std::move(x))` | std::move(x)是将亡值,返回T&&,即int&&        |

<ins>**表达式 (xxx) 本身的值类别通常是左值 (lvalue) (因为变量 xxx 是一个左值，加括号不改变其左值性)**</ins>

> [!TIP]
>  decltype(xxx) vs decltype((xxx)) (当 xxx 是变量名时):
> decltype(x): x 的声明类型。
> decltype((x)): 如果 x 是 T 类型并且是左值，结果是 T&。
> 表达式类型的定义: <ins>**表达式的类型总是非引用类型**</ins>
> 表达式类型的定义: <ins>**表达式 (y) 的类型就是被引用对象的类型（加上cv限定符）**</ins>

> [!NOTE]
> x是表达式,decltype((x)) 永远返回左值引用

```C++
int x = 10;
int* ptr = &x;
const int cx = 20;
int& z = x;
int&& w = 20;
```

| expression        | 结果                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------- |
| `decltype((x))`   | x是左值，返回int&                                                                       |
| `decltype((ptr))` | ptr是左值，返回int*&                                                                    |
| `decltype((cx))`  | cx是左值，返回const int&                                                                |
| `decltype((z))`   | z是左值（可以取地址）,类型是 int&，返回左值引用，即(int&)&， 根据引用折叠规则，即int&   |
| `decltype((w))`   | w是左值（可以取地址）,类型是 int&&，返回左值引用，即(int&&)&， 根据引用折叠规则，即int& |

# 引用折叠规则

<ins>**只要有一个左值引用，结果就是左值引用**</ins>
<ins>**只有当两个都是右值引用 && 时，结果才是右值引用 &&.**</ins>

| 初始引用 | 加上  | 结果  |
| -------- | ----- | ----- |
| `T&`     | `&`   | `T&`  |
| `T&`     | `&&	` | `T&`  |
| `T&&`    | `&`   | `T&`  |
| `T&&`    | `&&	` | `T&&` |


# declval 

## 定义

<ins>**在不实际创建对象的情况下，获得一个假设存在的对象的“引用”**</ins>

> [!TIP]
> 如果 T 是一个对象类型 (例如 int, MyClass)，declval<T>() 返回 T&& (右值引用)。
> 如果 T 是一个左值引用类型 (例如 int&, MyClass&)，declval<T>() 返回 T& (左值引用)。
> 如果 T 是一个右值引用类型 (例如 int&&, MyClass&&)，declval<T>() 返回 T&& (右值引用)。
> (注意：引用折叠规则 T& && -> T&, T&& && -> T&&)

## 场景

- 只能用于非求值上下文 (unevaluated context)：例如 sizeof, decltype, noexcept 的操作数，以及某些 SFINAE 场景。你不能在运行时实际调用 declval，因为它没有定义，链接时会出错。
- 无需构造函数：因为不实际创建对象，所以 T 不需要有可访问的构造函数 (默认构造、拷贝构造等)。这对于抽象类、没有默认构造函数的类等非常有用。


## 功能

- 模拟对象存在：允许你在编译时“假装”有一个类型 T 的对象，以便进行类型推导或特性检查
- 类型推导：与 decltype 结合使用，推断表达式的结果类型，特别是当表达式涉及成员函数调用或操作符重载时
- SFINAE (Substitution Failure Is Not An Error)：在模板元编程中，用于检查某个类型是否支持特定操作，或者某个成员是否存在
- noexcept 说明符：用于检查某个操作对于特定类型是否会抛出异常


> 如果一个类没有默认构造函数，我们如何在编译时获取其成员函数的返回类型？

```C++

class Widget {
  Widget(int, std::string) {} // 无默认构造函数
public:
  double compute() const;
};

decltype(Widget{}.compute()) x; // 错误：无法构造 Widget 对象

decltype(std::declval<Widget>().compute()) x; // x 的类型是 double
```

# 实践

## 完美转发返回类型

```C++
template <typename Container, typename Index>
static auto get_element(Container & c, Index i) -> decltype(c[i]) {
    return c[i];
}
```

## 保留精确类型（C++14 decltype(auto)）

```C++
template <typename T>
static auto forward_value(T && t) -> decltype(auto) {
    return std::forward<T>(t);
}
```

## 获取成员类型

```C++
template <typename T>
class MyContainer {
private:
    T container_;

public:
    // 使用 decltype 获取迭代器的准确类型
    using value_type = decltype(*container_.begin());
    using iterator = decltype(container_.begin());

    auto begin() -> iterator { return container_.begin(); }
};
```

## 泛型编程
```C++
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
    return t + u;
}
```

## 去除cv

```C++
template<typename T>
std::decay_t<decltype(t)> copy(T&& t) {
    return std::forward<T>(t);
} 
```

## 与 std::declval 结合

> 在没有实际对象的情况下，推导一个成员函数在某个类型上被调用后的返回类型

```C++
#include <utility>

struct Foo {
    int bar(double) { return 0; }
};
using BarReturnType = decltype(std::declval<Foo>().bar(std::declval<double>()));

```
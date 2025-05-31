
# decltype 

<ins>**decltype是一个关键字,而不是函数,用于在编译期推断一个表达式的类型,不会实际计算表达式的值**</ins>


> [!IMPORTANT]
> decltype(expression): 如果 expression 是一个未加括号的 id-expression (标识符表达式，如变量名) 或者一个未加括号的类成员访问表达式 ( object.member 或 pointer->member ), 返回该标识符或成员被声明时的类型 (declared type)
> - 例如，如果 int x = 0;，那么 decltype(x) 就是 int。
> - 如果 const int& y = x;，那么 decltype(y) 就是 const int&。
> - 如果 struct S { double m; }; S s;，那么 decltype(s.m) 就是 double

> [!IMPORTANT]
> `decltype((expression))`: 如果 `expression` 是任何其他类型的表达式 (例如，函数调用、字面量、带括号的标识符、算术运算等), 返回表达式的值类别 (value category) 和类型.
> - 如果 `expression` 的结果是一个左值 (lvalue)，类型为 T，那么 `decltype(expression)` 是 T& (左值引用)
> - 如果 `expression` 的结果是一个将亡值 (xvalue)，类型为 T，那么 `decltype(expression)` 是 T&& (右值引用)
> - 如果 `expression` 的结果是一个纯右值 (prvalue)，类型为 T，那么 `decltype(expression)` 是 T
> - 如果 `expression` 是一个函数，那么 `decltype(expression)` 是函数的返回值类型
> - 重要： `const 和 volatile` 限定符会被保留。

```c++
int x = 0;
const int& y = x;
```

| expression      | 结果                                                                                          |
| --------------- | --------------------------------------------------------------------------------------------- |
| `decltype(x)`   | `int`                                                                                         |
| `decltype(y)`   | `const int&`                                                                                  |
| `decltype((x))` | `int&`                                                                                        |
| `decltype((y))` | `(y)`是一个表达式, 类型是 const int., y本身是一个左值,按照规则,`decltype((y))` = `const int&` |

<ins>**表达式 (xxx) 本身的值类别通常是左值 (lvalue) (因为变量 xxx 是一个左值，加括号不改变其左值性)**</ins>

> [!IMPORTANT]
>  decltype(xxx) vs decltype((xxx)) (当 xxx 是变量名时):
> decltype(x): x 的声明类型。
> decltype((x)): 如果 x 是 T 类型并且是左值，结果是 T&。
> 表达式类型的定义: <ins>**表达式的类型总是非引用类型**</ins>
> 表达式类型的定义: <ins>**表达式 (y) 的类型就是被引用对象的类型（加上cv限定符）**</ins>

# declval 

## 定义

<ins>**在不实际创建对象的情况下，获得一个假设存在的对象的“引用”**</ins>

> [!IMPORTANT]
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
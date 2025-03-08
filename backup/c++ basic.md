# 构造函数

```c++
Object a; // 默认构造函数
Object b = a; // 拷贝构造函数
Object c(b);// 拷贝构造函数
c = a; // 赋值运算符
```

- **拷贝构造函数用于新对象的初始化**
- **赋值运算符用于已存在对象的赋值**
- **函数按值传参或返回对象时，也会触发拷贝构造函数或者移动构造函数**

# lvalue/rvalue

```c++
int f(std::string &str) { return 0; }
int f(const std::string &str) { return 1; }
int f(std::string &&str) { return 2; }

f("Hello!") -> 2
```

**重载解析优先级**

- 右值引用(**&&**): 优先匹配右值
- 常量左值引用(**const &**): 可接受右值，但是优先级低于右值引用
- 非const左值引用(**&**): 无法绑定右值

> `"Hello!` 是字符串字面量，即右值，优先匹配右值引用的函数

# 类型转换

```c++
struct B {
  int m_a;
};

struct A {
  int m_a;
};

A a{0};
B b = static_cast<B>(a); // 允许编译通过
```

## 用户自定义转换函数

> https://zh.cppreference.com/w/cpp/language/cast_operator

```c++
struct A {
  int m_a;

// 允许A 通过static_cast的方式转换为B
  explicit operator B() const {
      return B{m_a};
  }
};
```

- 语法
    - `operator 转换类型标识`
    - `explicit operator 转换类型标识`
    - `explicit ( 表达式 ) operator 转换类型标识`

## 构造函数

```c++
struct A {
  int m_a;
};

struct B {
  int m_a;
  explicit B(const A &aObj) : m_a(aObj.m_a) {}
};

B b = static_cast<B>(a);
```

- `static_cast` 支持基本类型之间的合法转换
- `static_cast` 非基本类型时，尝试寻找显示构造函数和用户自定义构造函数，两种不能同时存在

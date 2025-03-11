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

# `const`

- **const 就近原则，修饰的是const左边的东西，如果左边没有，就修饰右边的**
- **左定值，右定向: const 在`*`左边时，指针指向的值（内容）不可变，在`*`右边时，指针的指向不可改变**

## 左定值

```c++

struct Item {
  int id = 0;
  std::string name;
};

const Item *item_ptr = new Item(1, "hello");
```

**const 在`*`的左边，即值不可变**

- 不可变部分: *item_ptr

> 编译报错 `error: read-only variable is not assignable`

```c++
(*item_ptr).id = 2;
(*item_ptr).name = "world";
```
- 可变部分: item_ptr

> 下面的编译正常

```c++
Item item2(2, "world");
item_ptr = &item2;
```

## 右定向

```c++
Item *const item_ptr = new Item(3, "hello");
```
- 不可变部分: `Item *`，也就是不能修改指向

> 下面的编译报错 `error: cannot assign to variable 'item_ptr' with const-qualified type 'Item *const'`

```c++
Item item3(3, "world");
item_ptr = &item3; // 不能修改指向
```

- 可变部分: `*item_ptr`

```c++
(*item_ptr).id = 4;
(*item_ptr).name = "world2";

```



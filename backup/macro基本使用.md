# macro

- [macro](#macro)
  - [字符串化](#字符串化)
  - [连接](#连接)
  - [变参宏](#变参宏)
  - [条件编译](#条件编译)
- [macro vs function](#macro-vs-function)
  - [宏优点](#宏优点)
  - [宏缺点](#宏缺点)


> 一般我们在CR中不建议宏，因为无法做到类型安全以及可读性上比较差
> 但是在一些业务无关的代码上，比如配置解析，可以有效减少重复代码 
> 下面收集一些宏的常见使用 

## 字符串化

> 简单说就是在符号的前后加上双引号。 #hello -> "hello"

```cpp
#define STR_MACRO(x) #x

std::clog << STR_MACRO(hello) << '\n';
std::clog << STR_MACRO(world) << '\n';
std::clog << STR_MACRO(1024) << '\n';
std::clog << STR_MACRO(3.14) << '\n';
```

## 连接

> concat two symbol . eigen 库中大量使用 

```cpp
template <typename Type, typename Row, typename Col>
class Matrix {};

#define MAKE_TYPEDEFS_MACRO(Size, SizeSuffix) \
    /* MatrixX*/ \
    template <typename Type> \
    using Matrix##SizeSuffix = Matrix<Type, Size, Size>; \
    /*VectorX*/ \
    template <typename Type> \
    using Vector##SizeSuffix = Matrix<Type, Size, 1>; \
    /*RowVectorX*/ \
    template <typename Type> \
    using RowVector##SizeSuffix = Matrix<Type, 1, Size>;
```

## 变参宏

```cpp
#define PRINT_VALUES(...) printValues(__VA_ARGS__)
```

## 条件编译

```cpp
#ifdef DEBUG
// 在调试模式下执行的代码
#endif
```

# macro vs function 

## 宏优点

- 在宏处理阶段做文本替换，无运行时函数调用开销 
- 支持字符串化以及连接，灵活性更好 

## 宏缺点 

- 可读性和维护性： 由于宏仅是简单的文本替换，可能导致代码可读性较差，特别是在宏定义比较复杂的情况下。这也增加了代码维护的难度。
- 类型安全： 宏不具备类型检查，可能导致一些潜在的类型错误，而函数则在编译时会进行类型检查
# 元函数 

- 输入: 通过模板参数 
- 输出: 通常通过类型别名(using/typedef)或静态成员来提供 
- 计算过程: 编译期

```c++
template <typename T> // 输入 
struct is_pointer {
    static constexpr bool value = false; // 输出
};

template <typename T>
struct is_pointer<T *> {
    static constexpr bool value = true;
};

static_assert(is_pointer<int *>::value); // 计算过程
static_assert(!is_pointer<int>::value);
```

# 模板参数 

- 类型参数

```c++
template <typename T>
struct Container {
    T val;
};
```

- 非类型参数

> 必须是 编译期常量

```c++
template <int size>
struct Array {
    std::array<int, size> data;
};
```

- 模板参数 
```c++
template <template <typename> class Container>
class Wrapper {
    Container<int> data; // 使用传入的模板
public:
    void add(int value) { data.push_back(value); }
};
```


# `restrict` 关键字

## 作用

> 程序员向编译器保证，在指针的整个生命周期内，只有这个指针能用来访问它指向的对象。换句话说，没有其他指针会指向同一块内存区域。

```c
void add_arrays(int* a, int* b, int* result, int size);
```

编译器必须考虑，a/b/c 可能指向同一块内存，也就是是同一个指针的别名，那么每次使用都需要从内存里面重新加载这个值（因为值可能被其它指针修改）

```c
void add_arrays(int* restrict a, int* restrict b, int* restrict result, int size);
```

增加 `restrict` 修饰后,编译期认为这三个指针是独立的，可以进行更激进的优化，比如 将数据缓存到寄存器中/进行循环展开/使用 SIMD 指令进行并行计算等。

## 说明

- 主要用于函数参数，特别是涉及大量数据处理的函数。
- 只在确实能保证指针指向独立内存区域时使用。
- 在性能关键的代码中使用，因为这是 restrict 发挥作用的地方
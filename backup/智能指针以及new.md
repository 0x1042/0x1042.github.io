# 智能指针自定义deleter

- [智能指针自定义deleter](#智能指针自定义deleter)
  - [函数指针](#函数指针)
  - [函数对象](#函数对象)
  - [`lambda`函数](#lambda函数)
- [对象池](#对象池)
- [`new`](#new)
  - [`new operator` 是关键字](#new-operator-是关键字)
  - [`operator new` 是一个函数](#operator-new-是一个函数)
  - [`placement new` 是`operator new`的重载](#placement-new-是operator-new的重载)

## 函数指针 

```cpp
void deleteVec(int * ptr) {
    delete[] ptr;
}

TEST_CASE("test deleter", "[fb_pointer]") {
    std::unique_ptr<int, decltype(&deleteVec)> my_ptr(new int[5], deleteVec);
}
```
## 函数对象 

```cpp
template <typename T>
struct CustomDeleter {
    void operator()(T * t) const {
        delete t;
        std::clog << "CustomDeleter run... delete type is " << infra::Name<T>().name << '\n';
    }
};

TEST_CASE("test deleter", "[fb_obj]") {
    std::unique_ptr<int, CustomDeleter<int>> int_ptr(new int);
    std::unique_ptr<double, CustomDeleter<double>> double_ptr(new double);
}
```

## `lambda`函数

```cpp
TEST_CASE("test deleter", "[lambda]") {
    auto deleter = [](int * ptr) {
        delete[] ptr;
        std::clog << " delete by lambda ..." << '\n';
    };
    std::unique_ptr<int, decltype(deleter)> arr(new int[5], deleter);
}
```

# 对象池


- 要缓存的对象

```cpp
class Object {
public:
    Object(const Object &) = default;
    Object(Object &&) = delete;
    auto operator=(const Object &) -> Object & = default;
    auto operator=(Object &&) -> Object & = delete;
    explicit Object(int id) : id(id) { std::clog << "Object " << id << " is created.\n"; }

    ~Object() { std::cout << "Object " << id << " is destroyed.\n"; }

    void doSomething() const;

    [[nodiscard]] auto getId() const -> int;
    void setId(int id_);

private:
    int id;
};
```

- 定义对象池 

```cpp

// pool.h 
class ObjectPool {
public:
    struct ObjectDeleter {
        void operator()(Object * obj) {
            std::clog << "fake delete " << obj->getId() << '\n';
            obj->setId(0);
            pool.push_back(obj);
        }
    };

    static auto acquire() -> std::unique_ptr<Object, ObjectDeleter>;

    static auto size() -> size_t;

private:
    inline static std::vector<Object *> pool; // 对象池
};

// pool.cpp
auto ObjectPool::acquire() -> std::unique_ptr<Object, ObjectDeleter> {
    if (pool.empty()) {
        // return std::unique_ptr<Object, ObjectDeleter>(new Object(0), ObjectDeleter());
        return {new Object(0), ObjectDeleter()};
    }
    Object * obj = pool.back();
    pool.pop_back();
    // return std::unique_ptr<Object, ObjectDeleter>(obj, ObjectDeleter());
    return {obj, ObjectDeleter()};
}

auto ObjectPool::size() -> size_t {
    return pool.size();
}
```


# `new`

## `new operator` 是关键字 

就是`new` 关键字 `T *t = new T();` 背后包含两个动作

1. 调用 `operator new` 分配内存 ,并返回指向该对象的指针 
2. 调用`T`的构造函数 
3. 不可被重载 


## `operator new` 是一个函数 

是一个动态内存分配的函数 

```cpp
void* operator new(std::size_t size);
```

- 只分配所要求的空间，不调用相关对象的构造函数
- 当无法满足所要求分配的空间时
  - 如果有`new_handler`，则调用`new_handler`
  - 否则如果没要求不抛出异常（以nothrow参数表达），则执行bad_alloc异常，否则
  - 否则返回0
- 可以被重载，重载时，返回类型必须声明为`void*`
- 第一个参数类型必须为表达要求分配空间的大小（字节），类型为size_t,可以有其他参数


## `placement new` 是`operator new`的重载

> 原型

```cpp
void *operator new( size_t, void * p ) throw() { return p; }
```

1. 是重载`operator new`的一个标准、全局的版本，它不能够被自定义的版本代替(不像普通版本的operator new和operator delete能够被替换)
2. 执行忽略`size_t`参数，只返回第二个参数。
3. 其结果是允许用户把一个对象放到一个特定的地方，达到调用构造函数的效果
4. 和其他普通的`new`不同的是，它在括号里多了另外一个参数
5. 它并不分配内存，只是返回指向已经分配好的某段内存的一个指针。因此不能删除它，但需要调用对象的析构函数

```cpp
TEST_CASE("test deleter", "[operator new]") {
    // 分配内存
    char buffer[sizeof(pool::Object)];

    // 使用 placement new 在已分配的内存上构造对象
    pool::Object * obj = new (buffer) pool::Object(1024);

    // 注意：在使用了 placement new 后，应该手动调用对象的析构函数
    obj->~Object();
}
```

# 汇总对比

| expr               | 类型 | Syntax                     | 析构                 | 是否分配内存 | 是否构造对象 | 是否可重载 | 典型用途                 |
| ------------------ | ---- | -------------------------- | -------------------- | ------------ | ------------ | ---------- | ------------------------ |
| `new`              | 语法 | `auto* ptr = new T(args);` | `delete or delete[]` | 是           | 是           | 否         | 创建对象最常用方式       |
| `operator new`     | 函数 | `operator new(sizeof(T));` | `operator delete`    | 是           | 否           | 是         | 替换 malloc 逻辑，内存池 |
| 类内`operator new` | 函数 | `operator new(sizeof(T));` | `operator delete`    | 是           | 是           | 是         | 控制类对象的分配策略     |
| `placement new`    | 语法 | `T* p = new(buffer) T(args);`       | 手动调用`p->~T();`   | 否           | 否           | 否         | 在指定内存构造对象       |
# 静态变量

## 分类 

<img width="1262" height="1292" alt="Image" src="https://github.com/user-attachments/assets/d79c3188-bdec-497d-a230-22dcfcf75fc2" />

## 构造 

- 静态变量的构造发生在程序启动期间（main函数执行前）
- 同一翻译单元内的静态变量按照定义顺序构造
- 不同翻译单元间的静态变量构造顺序未定义（但C++11后提供了线程安全保证）
- 局部静态变量在首次执行到声明语句时构造 

## 析构 

- 与构造顺序相反 - 静态变量的析构顺序与构造顺序严格相反（LIFO，后进先出）
- 全局命名空间析构 - 所有静态变量的析构发生在main函数返回后，程序终止前
- 翻译单元局部顺序 - 同一翻译单元内的静态变量按照与定义相反的顺序析构
- 跨翻译单元顺序未定义 - 不同翻译单元间的静态变量析构顺序是未定义的

## 生命周期 

|类型/周期|生命周期|内存分配|构造|析构|
|----|----|----|----|----|
|全局/命名空间静态变量|从程序启动到程序结束|在静态存储区分配|在main函数执行前|在main函数返回后，程序终止前|
|类静态成员变量|从程序启动到程序结束（即使没有类的实例）|在静态存储区分配|在main函数执行前（或在首次访问前，取决于实现）|在main函数返回后，程序终止前|
|局部静态变量|首次执行到声明语句时到程序结束|在静态存储区分配|在首次执行到声明语句时（C++11后保证线程安全）|在main函数返回后，程序终止前|

# 退出 

<img width="2090" height="1532" alt="Image" src="https://github.com/user-attachments/assets/4dc6b233-e20d-4cb0-97f4-ce218d46154f" />

# exit 函数

|特性/方法|`return 0`|`std::exit(0)`|`std::quick_exit(0)`|`_exit(0) / _Exit(0)`|
|----|----|----|----|----|
|局部对象析构|✅|❌|❌|❌|
|static/全局对象析构|✅|✅|❌|❌|
|atexit() 回调|✅|✅|❌|❌|
|at_quick_exit() 回调|❌|❌|✅|❌|
|栈展开|✅|❌|❌|❌|
|缓冲区刷新|✅|✅|✅|❌|
|临时文件清理|✅|✅|可能|❌|
|是否可能 coredump|可能|可能|❌|❌|
|退出速度|慢|慢|极快|极快|
|标准|c++|c++|c++|POSIX/C99|

## 使用建议 

|方法|场景|
|----|----|
|`return 0`|绝大部分场景|
|`std::exit(0)`|深层嵌套函数中需要立即终止程序|
|`std::quick_exit(0)`|需要非常快速的退出（避免大量析构函数的开销<br>多线程程序中避免析构死锁<br>已知程序状态不稳定，不安全执行析构|
|`_exit(0)`|需要立即终止，不能有任何延迟<br>完全不清理任何资源<br>最暴力的终止方式|

比如，大量的静态变量造成的析构顺序不确定,容易触发core，这种情况下，使用`std::quick_exit(0)`或者`_exit(0)`即可


# `__attribute__((destructor))` 与 `__attribute__((constructor))`


## `__attribute__((destructor))` 

<img width="2436" height="3064" alt="Image" src="https://github.com/user-attachments/assets/020ba5f8-3169-4f0c-afd0-b79c7575e5b6" />

- ` __attribute__((constructor(0-100)))` 系统保留
- ` __attribute__((constructor(101))) ` 最高优先级用户代码
- `__attribute__((constructor))`        无优先级（默认）


## `__attribute__((constructor))`

<img width="1824" height="3328" alt="Image" src="https://github.com/user-attachments/assets/f19b90d8-fe64-4f8c-8c5a-920f1ce601b4" />

- `std::quick_exit() ` 和 `_exit()` 场景不会调用 







# 值类别 

- [值类别](#值类别)
  - [如何区分左值和右值](#如何区分左值和右值)
  - [函数参数匹配](#函数参数匹配)
  - [值类型](#值类型)
  - [万能引用 **universal reference**](#万能引用-universal-reference)
  - [引用折叠 **reference collapsing**](#引用折叠-reference-collapsing)
  - [完美转发 **perfect forward**](#完美转发-perfect-forward)

## 如何区分左值和右值

```c++
void log(std::string_view message, std::source_location location) {
    std::clog << "file: " << location.file_name() << '(' << location.line() << ':' << location.column() << ") `" << location.function_name()
              << "`: " << message << '\n';
}

void foo(int & /*val*/) {
    log("foo1");
}

void foo(int && /*val*/) {
    log("foo2");
}
```

```cpp
int && value = 1024;
foo(value); // 调用的是 void foo(int &)
```

- 匿名的临时对象是右值，具名的右值引用对象是左值 
- 如果表达式可以取地址，则为左值表达式，否则，为右值表达式
- 表达式 value 是具名的右值引用对象，value 也可以取地址，所以 表达式value 是一个左值，匹配第一个函数

## 函数参数匹配

| 参数类型      | 说明                                     |
| ------------- | ---------------------------------------- |
| Value&        | 只能匹配左值表达式                       |
| Value&&       | 只能绑定右值表达式（模板函数下单独讨论） |
| const Value&  | 可以匹配左值和右值表达式                 |
| const Value&& | 实际不使用                               |


## 值类型 

- 泛左值： 左值 和 将亡值
- 右值：纯右值 和 将亡值
- `static_cast<Value&&>(value)` 是将亡值，常见的将亡值是 函数的返回值 
![types](https://github.com/0x1042/0x1042.github.io/assets/7525242/0fae6f7f-bce8-41b6-ad2d-45cc312ec7b4)


## 万能引用 **universal reference** 

> 如何区分 Arg&& 是右值引用还是万能引用？

- **如果 Arg&& 是模板参数或者 auto，则是万能引用，「既可以接受左值，也可以接受右值」，否则为右值引用**
- **万能引用在类型推导语境下，可以保留类型的cv限定符「const和volatile」和值类别**

## 引用折叠 **reference collapsing**

> 为了解决 reference to reference 的问题

c++ 中不允许指向引用的引用，对于指向引用的引用会被简化，推导规则如下 

```cpp
template <typename T>
void Example(T && input) {}
```

- 函数形参是左值，T&，传入的实参是左值，即input 是 `T& &`
- 函数形参是左值，T&，传入的实参是右值，即input 是 `T& &&`
- 函数形参是右值，T&&，传入的实参是左值，即input 是 `T&& &`
- 函数形参是右值，T&&，传入的实参是右值，即input 是 `T&& &&`

推导规则是： **仅当两个都是右值引用时，推导为右值，其余情况为左值**，也就是 `T&& &&`为右值，其他情况为左值 


## 完美转发 **perfect forward** 

> 在传参的过程中，保留参数的原始类型 

```cpp
void foo(int & val);
void foo(int && val);

template <typename T> void call_foo(T && t) {
    foo(std::forward<T>(t));
}
```

如何实现的？

```cpp
template <class _Tp>
_LIBCPP_NODISCARD_EXT inline _LIBCPP_HIDE_FROM_ABI _LIBCPP_CONSTEXPR _Tp&&
forward(_LIBCPP_LIFETIMEBOUND __libcpp_remove_reference_t<_Tp>& __t) _NOEXCEPT {
  return static_cast<_Tp&&>(__t);
}

template <class _Tp>
_LIBCPP_NODISCARD_EXT inline _LIBCPP_HIDE_FROM_ABI _LIBCPP_CONSTEXPR _Tp&&
forward(_LIBCPP_LIFETIMEBOUND __libcpp_remove_reference_t<_Tp>&& __t) _NOEXCEPT {
  static_assert(!is_lvalue_reference<_Tp>::value, "cannot forward an rvalue as an lvalue");
  return static_cast<_Tp&&>(__t);
}
```

- Tp 是左值， `static_cast<_Tp&&>(__t)` 后是 T& &&，按照引用折叠规则，`T& && -> T&`, Tp是左值 
- Tp 是右值， `static_cast<_Tp&&>(__t)` 后是 T&& &&，按照引用折叠规则，`T&& && -> T&&`, Tp是右值 
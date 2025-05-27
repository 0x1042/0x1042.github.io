
# trait

## 定义

- 是一个<ins>**类模板**</ins>,通常是<ins>**struct**</ins>
- 用于在<ins>**编译时获取或者查询某个类型T的信息或者属性**</ins>
- 不包含运行时数据,通常通过<ins>**static constexpr 成员**</ins>或者<ins>**嵌套类型别名(using type = ...)**</ins>


## 功能

- 类型信息查询. 比如<ins>**`std::is_integral<T>`**</ins> 查询`T`是否是整形
- 类型转换/关联. 比如<ins>**`std::remove_pointer<T*>::type`**</ins> 移除`T`的指针属性
- 条件编译. 与 <ins>**std::enable_if 或 if constexpr**</ins> 结合,根据类型特性选择不同的代码路径

## 原理

- 主模板 (Primary Template): <ins>**定义通用的情况或默认值**</ins>
- 模板特化 (Template Specialization): 为<ins>**特定类型或满足特定条件的类型提供特定的信息**</ins>


## 🌰

```c++

// step 1
// 主模板 定义通用情况
template <typename T>
struct is_my_void {
  static constexpr bool value = false;
};

// step 2
// 特化模板 定义特例情况
template <> struct
is_my_void<void> {
  static constexpr bool value = true;
};

// step3 可选 定义辅助变量
template <typename T>
inline constexpr bool is_my_void_v = is_my_void<T>::value;
```

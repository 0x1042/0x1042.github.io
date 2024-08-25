# c兼容

```c++
#pragma once

#ifdef __cplusplus

extern "C" {
#endif
void parse_entry(char *fname, int level);

#ifdef __cplusplus
}

#endif
```

# cmake 

```cmake
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)

add_library(parse STATIC ${LIBSRC})
```


# rust 32 bit target

```

# add target
rustup target add i686-unknown-linux-gnu

# 编译
cargo build --target=i686-unknown-linux-gnu
```

# 导入

## 依赖

```rust
[build-dependencies]
cc = "1.0"
```

## 代码 

```rust 
#[link(name = "parse", kind = "static")]
extern "C" {
    fn parsev2() -> c_void;
}
```

## build.rs

```rust
fn main() {
    println!("cargo:rustc-link-search=native=reallibpath");
}

```

## 编译失败 
<img width="1876" alt="image" src="https://github.com/user-attachments/assets/e2121d3b-f342-490f-bf15-e19141aaae35">

```
undefined reference to std::__cxx11::basic_string<char, std::char_traits<char>, 
std::allocator<char> >::rfind(char, unsigned int) const
```

> 更新build.rs

```rust
// build.sh 

fn main() {
    // 链接c++标准库和gcc动态库 
    println!("cargo:rustc-link-lib=dylib=stdc++");
    println!("cargo:rustc-link-lib=dylib=gcc_s");
}
```

> 或者全静态编译lib

```cmake
target_link_libraries(parse
    -static-libstdc++
    -static-libgcc
)
```


# 32位编译 m32 

```
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
```

# 安装32位lib 

```
sudo apt install gcc-multilib g++-multilib libc6-dev-i386 -y
```

# 开启O3

> `cmake` 默认`RelWithDebInfo` 模式是`O2`

```cmake
set(CMAKE_C_FLAGS_RELWITHDEBINFO
    "-O3 -g"
    CACHE STRING
          "Flags used by the C compiler during release builds with debug info."
          FORCE)
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO
    "-O3 -g"
    CACHE
      STRING
      "Flags used by the C++ compiler during release builds with debug info."
      FORCE)
```
 




- [定义](#定义)
- [`copy constructor`](#copy-constructor)
  - [问题](#问题)
  - [解决方案](#解决方案)
- [`operator=`](#operator)
  - [问题](#问题-1)
  - [解决方案](#解决方案-1)
- [`move constructor`](#move-constructor)
  - [问题](#问题-2)
  - [解决方案](#解决方案-2)
- [`move operator`](#move-operator)
  - [问题](#问题-3)
  - [解决方案](#解决方案-3)

# 定义 

```cpp
class SString {
public:
    explicit SString(const char * cp);

    virtual ~SString();

private:
    char * data_;
};

// impl 
// 确保长度
SString::SString(const char * cp) : data_(new char[strlen(cp) + 1]) {
    strcpy(data_, cp);
}

SString::~SString() {
    delete[] data_;
}
```

> the rule of five 
> - Destructor
> - Copy Constructor
> - Copy Assignment Operator
> - Move Constructor
> - Move Assignment Operator


# `copy constructor`

> 考虑一下使用方式 ，这个代码有两个问题 

```cpp

void foo(rof::SString val) {
    spdlog::info("size is {}", val.size());
}

TEST_CASE("case1", "copy") {
    rof::SString s{"hello world"};
    { foo(s); }
    { foo(s); }
}
```

```
==23702==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000037f0 at pc 0x000104cd721d bp 0x7ff7bc0ea590 sp 0x7ff7bc0e9d58
READ of size 3 at 0x6020000037f0 thread T0
    #0 0x104cd721c in strlen+0x80c (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0x1b21c)
    #1 0x103fc4869 in tests::foo(rof::SString)+0x181 (c20:x86_64+0x1001b1869)
    #2 0x103fc4c64 in tests::CATCH2_INTERNAL_TEST_0()+0x2fc (c20:x86_64+0x1001b1c64)
    #3 0x103eadd03 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009ad03)
    #4 0x103ead79a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x10009a79a)
    #5 0x103e8f870 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007c870)
    #6 0x103e8e9e4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007b9e4)
    #7 0x103f6702c in main+0x624 (c20:x86_64+0x10015402c)
    #8 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

0x6020000037f0 is located 0 bytes inside of 12-byte region [0x6020000037f0,0x6020000037fc)
freed by thread T0 here:
    #0 0x104dad5ad in _ZdaPv+0x7d (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf15ad)
    #1 0x103fc4bfc in tests::CATCH2_INTERNAL_TEST_0()+0x294 (c20:x86_64+0x1001b1bfc)
    #2 0x103eadd03 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009ad03)
    #3 0x103ead79a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x10009a79a)
    #4 0x103e8f870 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007c870)
    #5 0x103e8e9e4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007b9e4)
    #6 0x103f6702c in main+0x624 (c20:x86_64+0x10015402c)
    #7 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

previously allocated by thread T0 here:
    #0 0x104dad19d in _Znam+0x7d (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf119d)
    #1 0x103fc4a6d in tests::CATCH2_INTERNAL_TEST_0()+0x105 (c20:x86_64+0x1001b1a6d)
    #2 0x103eadd03 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009ad03)
    #3 0x103ead79a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x10009a79a)
    #4 0x103e8f870 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007c870)
    #5 0x103e8e9e4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007b9e4)
    #6 0x103f6702c in main+0x624 (c20:x86_64+0x10015402c)
    #7 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

SUMMARY: AddressSanitizer: heap-use-after-free (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0x1b21c) in strlen+0x80c
```

## 问题
1. use after free
2. double free 

## 解决方案 
> 实现copy constructor，避免两个指针指向一块内存地址

```cpp 

SString(const SString & other);

// impl
SString::SString(const SString & other) : data_(new char[strlen(other.data_) + 1]) {
    strcpy(data_, other.data_);
}

```

# `operator=`

> 考虑一下使用方式

```cpp
TEST_CASE("sstring2", "operator") {
    rof::SString hello{"hello"};
    rof::SString world{"world"};

    world = hello;
}
```

```
==24524==ERROR: AddressSanitizer: attempting double-free on 0x602000003830 in thread T0:
    #0 0x1061ad5ad in _ZdaPv+0x7d (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf15ad)
    #1 0x105424859 in tests::CATCH2_INTERNAL_TEST_2()+0x2b1 (c20:x86_64+0x1001b1859)
    #2 0x10530d303 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009a303)
    #3 0x10530cd9a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x100099d9a)
    #4 0x1052eee70 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007be70)
    #5 0x1052edfe4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007afe4)
    #6 0x1053c662c in main+0x624 (c20:x86_64+0x10015362c)
    #7 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

0x602000003830 is located 0 bytes inside of 6-byte region [0x602000003830,0x602000003836)
freed by thread T0 here:
    #0 0x1061ad5ad in _ZdaPv+0x7d (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf15ad)
    #1 0x1054247f2 in tests::CATCH2_INTERNAL_TEST_2()+0x24a (c20:x86_64+0x1001b17f2)
    #2 0x10530d303 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009a303)
    #3 0x10530cd9a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x100099d9a)
    #4 0x1052eee70 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007be70)
    #5 0x1052edfe4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007afe4)
    #6 0x1053c662c in main+0x624 (c20:x86_64+0x10015362c)
    #7 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

previously allocated by thread T0 here:
    #0 0x1061ad19d in _Znam+0x7d (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf119d)
    #1 0x105423d2f in rof::SString::SString(char const*)+0x4f (c20:x86_64+0x1001b0d2f)
    #2 0x10542467a in tests::CATCH2_INTERNAL_TEST_2()+0xd2 (c20:x86_64+0x1001b167a)
    #3 0x10530d303 in Catch::RunContext::runCurrentTest(std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>&)+0x203 (c20:x86_64+0x10009a303)
    #4 0x10530cd9a in Catch::RunContext::runTest(Catch::TestCaseHandle const&)+0x1ea (c20:x86_64+0x100099d9a)
    #5 0x1052eee70 in Catch::Session::runInternal()+0xe20 (c20:x86_64+0x10007be70)
    #6 0x1052edfe4 in Catch::Session::run()+0x44 (c20:x86_64+0x10007afe4)
    #7 0x1053c662c in main+0x624 (c20:x86_64+0x10015362c)
    #8 0x7ff811187365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

SUMMARY: AddressSanitizer: double-free (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf15ad) in _ZdaPv+0x7d
```

## 问题 
- double free

> 赋值之后，world 和 hello 的指针都指向了 hello的指针的内存地址
> 析构的时候，会析构两次
 
## 解决方案 

> 实现自定义的 operator= (copy assignment operator)

```cpp

auto operator=(const SString & other) -> SString &;

auto SString::operator=(const SString & other) -> SString & {
    // bugprone-unhandled-self-assignment
    if (this == &other) {
        return *this;
    }

    char * newdata = new char[strlen(other.data_) + 1];
    strcpy(newdata, other.data_);
    std::swap(newdata, this->data_);
    delete[] newdata;
    return *this;
}
```

# `move constructor`

> 用法 

```cpp 
TEST_CASE("sstring3", "move1") {
    rof::SString hello{"hello"};

    foo(std::move(hello));
}
```

## 问题

```
Passing result of std::move() as a const reference argument; no move will actually happen (fix available)clang-tidyperformance-move-const-arg
ruleoffive.h(6, 7): 'SString' is not move assignable/constructible
```

## 解决方案

```cpp
SString(SString && other) noexcept;

// impl
SString::SString(SString && other) noexcept : data_(other.data_) {
    other.data_ = nullptr;
}
```

# `move operator` 

> 用法 

```cpp 
TEST_CASE("sstring4", "move2") {
    rof::SString hello{"hello"};
    rof::SString world{"world"};

    world = std::move(hello);
}
```

## 问题

```
no move will actually happen (fix available)clang-tidyperformance-move-const-arg
ruleoffive.h(6, 7): 'SString' is not move assignable
```

## 解决方案

```cpp
auto operator=(SString && other) noexcept -> SString &;

auto SString::operator=(SString && other) noexcept -> SString & {
    delete[] data_;
    data_ = other.data_;
    other.data_ = nullptr;
    return *this;
}
```
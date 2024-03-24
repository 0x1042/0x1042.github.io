
- [配置](#配置)
- [测试](#测试)
- [编译运行](#编译运行)
  - [`-fsanitize=thread`](#-fsanitizethread)
  - [`-fsanitize=address`](#-fsanitizeaddress)
- [`sanitier`](#sanitier)

# 配置 

```cmake
option(ENABLE_SANITIZE "Enable feature" ON)

if(ENABLE_SANITIZE) 
  add_compile_options("-fno-omit-frame-pointer") 
  add_compile_options("-fno-optimize-sibling-calls")
  # add_compile_options("-fsanitize=address")
  add_compile_options("-fsanitize=thread")
  add_compile_options("-fsanitize-address-use-after-scope")
  # set(CMAKE_EXE_LINKER_FLAGS "-fno-omit-frame-pointer -fno-optimize-sibling-calls -fsanitize=address" ) 
  set(CMAKE_EXE_LINKER_FLAGS "-fno-omit-frame-pointer -fno-optimize-sibling-calls -fsanitize=thread" ) 
endif()
```

# 测试

```c++
#include <functional>
#include <string>
#include <thread>
#include <vector>

#include <fmt/ranges.h>
#include <gtest/gtest.h>

void add_to_vec(std::vector<std::string> &target, const std::string &name) {
	for (int i = 0; i < 10; i++) {
		target.push_back(fmt::format("{}:{}", name, i));
	}
}

TEST(race, addtovec) {
	std::vector<std::string> dst;
	std::string hello = "hello";
	std::string world = "world";
	std::thread t1(add_to_vec, std::ref(dst), std::cref(hello));
	std::thread t2(add_to_vec, std::ref(dst), std::cref(world));
	t1.join();
	t2.join();
	std::cerr << " res: " << fmt::format("{}", dst) << '\n';
}
```

# 编译运行 

## `-fsanitize=thread`

```shell
WARNING: ThreadSanitizer: data race (pid=77881)
  Read of size 8 at 0x7ff7ba7ae880 by thread T4:
    #0 add_to_vec(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&) race.cc:14 (singleton:x86_64+0x100021ba9)
    #1 void* std::__1::__thread_proxy[abi:ue170006]<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct>>, void (*)(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&), std::__1::reference_wrapper<std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>>, std::__1::reference_wrapper<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const>>>(void*) thread.h:232 (singleton:x86_64+0x100022437)

  Previous write of size 8 at 0x7ff7ba7ae880 by thread T3:
    #0 add_to_vec(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&) race.cc:14 (singleton:x86_64+0x100021b0b)
    #1 void* std::__1::__thread_proxy[abi:ue170006]<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct>>, void (*)(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&), std::__1::reference_wrapper<std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>>, std::__1::reference_wrapper<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const>>>(void*) thread.h:232 (singleton:x86_64+0x100022437)
```

## `-fsanitize=address`

```shell
==84486==ERROR: AddressSanitizer: attempting free on address which was not malloc()-ed: 0x603000003058 in thread T4
    #0 0x10f6dd4ad in _ZdlPv (/usr/local/Cellar/llvm/17.0.6_1/lib/clang/17/lib/darwin/libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf14ad)
    #1 0x10eb5162e in add_to_vec(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10004162e)
    #2 0x10eb52fab in void* std::__1::__thread_proxy[abi:ue170006]<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct>>, void (*)(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>&, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const&), std::__1::reference_wrapper<std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>>>, std::__1::reference_wrapper<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>> const>>>(void*) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x100042fab)
    #3 0x10f6c59dd in asan_thread_start(void*) (/usr/local/Cellar/llvm/17.0.6_1/lib/clang/17/lib/darwin/libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xd99dd)
    #4 0x7ff819cf818a in _pthread_start (/usr/lib/system/libsystem_pthread.dylib:x86_64+0x618a)
    #5 0x7ff819cf3ae2 in thread_start (/usr/lib/system/libsystem_pthread.dylib:x86_64+0x1ae2)

0x603000003058 is located 40 bytes after 32-byte region [0x603000003010,0x603000003030)
freed by thread T0 here:
    #0 0x10f6dd4ad in _ZdlPv (/usr/local/Cellar/llvm/17.0.6_1/lib/clang/17/lib/darwin/libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf14ad)
    #1 0x10eb4e399 in tf::Executor::~Executor() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10003e399)
    #2 0x10eb142b9 in test_task_flow() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x1000042b9)
    #3 0x10eb17228 in taskflow_dag_Test::TestBody() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x100007228)
    #4 0x10eba18ea in void testing::internal::HandleExceptionsInMethodIfSupported<testing::Test, void>(testing::Test*, void (testing::Test::*)(), char const*) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x1000918ea)
    #5 0x10eba183f in testing::Test::Run() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10009183f)
    #6 0x10eba2b9f in testing::TestInfo::Run() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x100092b9f)
    #7 0x10eba3bc6 in testing::TestSuite::Run() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x100093bc6)
    #8 0x10ebb40ed in testing::internal::UnitTestImpl::RunAllTests() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x1000a40ed)
    #9 0x10ebb391a in bool testing::internal::HandleExceptionsInMethodIfSupported<testing::internal::UnitTestImpl, bool>(testing::internal::UnitTestImpl*, bool (testing::internal::UnitTestImpl::*)(), char const*) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x1000a391a)
    #10 0x10ebb389c in testing::UnitTest::Run() (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x1000a389c)
    #11 0x10eb57c2f in main (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x100047c2f)
    #12 0x7ff81996c365  (/usr/lib/dyld:x86_64+0xfffffffffff5c365)

previously allocated by thread T1 here:
    #0 0x10f6dd08d in _Znwm (/usr/local/Cellar/llvm/17.0.6_1/lib/clang/17/lib/darwin/libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xf108d)
    #1 0x10eb1efc9 in std::__1::pair<std::__1::__hash_iterator<std::__1::__hash_node<std::__1::__hash_value_type<std::__1::__thread_id, unsigned long>, void*>*>, bool> std::__1::__hash_table<std::__1::__hash_value_type<std::__1::__thread_id, unsigned long>, std::__1::__unordered_map_hasher<std::__1::__thread_id, std::__1::__hash_value_type<std::__1::__thread_id, unsigned long>, std::__1::hash<std::__1::__thread_id>, std::__1::equal_to<std::__1::__thread_id>, true>, std::__1::__unordered_map_equal<std::__1::__thread_id, std::__1::__hash_value_type<std::__1::__thread_id, unsigned long>, std::__1::equal_to<std::__1::__thread_id>, std::__1::hash<std::__1::__thread_id>, true>, std::__1::allocator<std::__1::__hash_value_type<std::__1::__thread_id, unsigned long>>>::__emplace_unique_key_args<std::__1::__thread_id, std::__1::piecewise_construct_t const&, std::__1::tuple<std::__1::__thread_id&&>, std::__1::tuple<>>(std::__1::__thread_id const&, std::__1::piecewise_construct_t const&, std::__1::tuple<std::__1::__thread_id&&>&&, std::__1::tuple<>&&) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10000efc9)
    #2 0x10eb1dfa2 in tf::Executor::_spawn(unsigned long)::'lambda'(tf::Worker&, std::__1::mutex&, std::__1::condition_variable&, unsigned long&)::operator()(tf::Worker&, std::__1::mutex&, std::__1::condition_variable&, unsigned long&) const (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10000dfa2)
    #3 0x10eb1db2c in void* std::__1::__thread_proxy[abi:ue170006]<std::__1::tuple<std::__1::unique_ptr<std::__1::__thread_struct, std::__1::default_delete<std::__1::__thread_struct>>, tf::Executor::_spawn(unsigned long)::'lambda'(tf::Worker&, std::__1::mutex&, std::__1::condition_variable&, unsigned long&), std::__1::reference_wrapper<tf::Worker>, std::__1::reference_wrapper<std::__1::mutex>, std::__1::reference_wrapper<std::__1::condition_variable>, std::__1::reference_wrapper<unsigned long>>>(void*) (/Users/weixuan/code/ccode/singleton/_build/singleton:x86_64+0x10000db2c)
    #4 0x10f6c59dd in asan_thread_start(void*) (/usr/local/Cellar/llvm/17.0.6_1/lib/clang/17/lib/darwin/libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xd99dd)
    #5 0x7ff819cf818a in _pthread_start (/usr/lib/system/libsystem_pthread.dylib:x86_64+0x618a)
    #6 0x7ff819cf3ae2 in thread_start (/usr/lib/system/libsystem_pthread.dylib:x86_64+0x1ae2)
```


# `sanitier`

| sanitizer   | 官方地址                                                    | 开启方式                | 说明             |
| ----------- | ----------------------------------------------------------- | ----------------------- | ---------------- |
| `thread`    | https://clang.llvm.org/docs/ThreadSanitizer.html.           | `-fsanitize=thread`     | 检测datarace     |
| `address`   | https://clang.llvm.org/docs/AddressSanitizer.html           | `-fsanitize=address`    | 内存错误         |
| `memory`    | https://clang.llvm.org/docs/MemorySanitizer.html            | `-fsanitize=memory`     | 检测未初始化读取 |
| `undefined` | https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html | ` -fsanitize=undefined` | 检测为定义行为   |


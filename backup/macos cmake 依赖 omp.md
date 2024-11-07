
# 配置

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
link_directories(/usr/local/opt/llvm/lib/c++)

find_package(OpenMP COMPONENTS CXX)
target_link_libraries(
  ${PROJECT_NAME}
  PUBLIC OpenMP::OpenMP_CXX
  PUBLIC c++abi
  PUBLIC c++
)
```

# 使用

```c++
auto batch_get2(const std::vector<std::string> & urls, std::vector<std::string> & resps) -> int {
    std::chrono::high_resolution_clock::time_point start = std::chrono::high_resolution_clock::now();
    resps.resize(urls.size());

#pragma omp parallel for schedule(dynamic, 1)
    for (int i = 0; i < urls.size(); i++) {
        get(urls[i], resps[i]);
    }
    std::chrono::high_resolution_clock::time_point finish = std::chrono::high_resolution_clock::now();

    spdlog::info("omp cost {}", std::chrono::duration_cast<std::chrono::milliseconds>(finish - start).count());

    return 0;
}
```

# SIMD Dot Product

## 实现

```cpp
#include <immintrin.h>
#include <pmmintrin.h>
#include <x86intrin.h>
#include <xmmintrin.h>

/**
 * @brief base 版本
 *
 * @param a
 * @param b
 * @param dim
 * @return float
 */
float dot(const void *a, const void *b, size_t dim) {
  float sum = 0;
  for (unsigned i = 0; i < dim; i++) {
    sum += ((float *)a)[i] * ((float *)b)[i];
  }
  return sum;
}

/**
 * @brief simd 版本
 *
 * @param a
 * @param b
 * @param dim
 * @return float
 */
float dot_simd(const void *a, const void *b, size_t dim) {

  float *va = (float *)a;
  float *vb = (float *)b;

  size_t qty16 = dim / 16;

  const float *enda = va + 16 * qty16;

  // 初始化一个零向量 长度为8
  __m256 sum = _mm256_set1_ps(0);

  while (va < enda) {
      // 将va的前8个元素加载到向量v1中
    __m256 v1 = _mm256_loadu_ps(va);
    va += 8;
    // 将vb的前8个元素加载到向量v2中
    __m256 v2 = _mm256_loadu_ps(vb);
    vb += 8;

    // 执行乘法和加法：sum += v1 * v2
    sum = _mm256_fmadd_ps(v1, v2, sum);

    // 将va的前8个元素加载到向量v1中
    v1 = _mm256_loadu_ps(va);
    va += 8;
    // 将vb的前8个元素加载到向量v2中
    v2 = _mm256_loadu_ps(vb);
    vb += 8;

    // 执行乘法和加法：sum += v1 * v2
    sum = _mm256_fmadd_ps(v1, v2, sum);
  }

    // 从256位向量中提取两个128位部分
  __m128 low = _mm256_extractf128_ps(sum, 0);// 低128位
  __m128 hight = _mm256_extractf128_ps(sum, 1); // 高128位


 // 将两个128位向量相加，得到一个包含4个float的向量
  __m128 sum128 = _mm_add_ps(low, hight);

  // 这时sum128包含4个float值：[a, b, c, d]

  // 第一次hadd：将相邻元素两两相加
  // 输入：[a, b, c, d]
  // 输出：[a+b, c+d, a+b, c+d]
  sum128 = _mm_hadd_ps(sum128, sum128);

  // 第二次hadd：再次将相邻元素两两相加
  // 输入：[a+b, c+d, a+b, c+d]
  // 输出：[a+b+c+d, a+b+c+d, a+b+c+d, a+b+c+d]
  sum128 = _mm_hadd_ps(sum128, sum128);

  // 处理剩余部分
  float tail_sum = 0.0f;
  size_t remainder = dim % 16;
  for (size_t i = 0; i < remainder; ++i) {
    tail_sum += va[i] * vb[i];
  }

 //_mm_cvtss_f32 将结果从SIMD寄存器提取到普通浮点数
  return _mm_cvtss_f32(sum128) + tail_sum;
}

```

## 编译

```meson
add_global_arguments('-march=native', language: 'cpp')
add_global_arguments('-mavx2', language: 'cpp')
add_global_arguments('-mpclmul', language: 'cpp')
add_global_arguments('-mbmi', language: 'cpp')

dot_inc = include_directories('.')

dot_src = files(
    'dot.cc',
)

dot = library(
    'dot',
    dot_src,
    dependencies: [
        random_dep,
    ],
    include_directories: dot_inc,
)

dot_dep = declare_dependency(
    include_directories: dot_inc,
    link_with: dot,
)
```

## unittest

### 实现

```c++
#include "dot.h"
#include <gtest/gtest.h>

TEST(check, check1) {
  constexpr size_t dim = 128;

  const auto &va = random(dim);
  const auto &vb = random(dim);

  const auto &base = dot(va.data(), vb.data(), dim);
  const auto &simd = dot_simd(va.data(), vb.data(), dim);

  EXPECT_NE(base, 0);
  EXPECT_NE(simd, 0);
  EXPECT_DOUBLE_EQ(base, simd);
}

TEST(check, check2) {
  constexpr size_t dim = 333;

  const auto &va = random(dim);
  const auto &vb = random(dim);

  const auto &base = dot(va.data(), vb.data(), dim);
  const auto &simd = dot_simd(va.data(), vb.data(), dim);

  EXPECT_NE(base, 0);
  EXPECT_NE(simd, 0);
  EXPECT_DOUBLE_EQ(base, simd);
}

int main(int argc, char **argv) {
  testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}

```

### 编译

```meson
executable(
    'ut',
    'ut.cc',
    install: true,
    dependencies: [thread_dep, random_dep, gtest_dep, dot_dep],
)
```

## benchmark

### 实现
```c++
#include "dot.h"
#include <benchmark/benchmark.h>

static void BM_dot(benchmark::State &state) {
  constexpr size_t dim = 128;

  const auto &va = random(dim);
  const auto &vb = random(dim);

  for (auto _ : state) {
    const auto &simd = dot_simd(va.data(), vb.data(), dim);
    benchmark::DoNotOptimize(simd);
  }
}

BENCHMARK(BM_dot);

int main(int argc, char **argv) {
  benchmark::Initialize(&argc, argv);
  benchmark::RunSpecifiedBenchmarks();
  return 0;
}

```

### 编译

```meson
executable(
    'bench',
    'bench.cc',
    install: true,
    dependencies: [thread_dep, random_dep, bench_dep, dot_dep],
)
```

``` text
./target/bench --benchmark_min_time=10s
2025-02-19T01:19:44+08:00
Running ./target/bench
Run on (16 X 2300 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB
  L1 Instruction 32 KiB
  L2 Unified 256 KiB (x8)
  L3 Unified 16384 KiB
Load Average: 1.89, 2.17, 2.30
------------------------------------------------------
Benchmark            Time             CPU   Iterations
------------------------------------------------------
BM_dot            94.0 ns         94.0 ns    147363798
BM_dot_simd       10.3 ns         10.2 ns   1442614248
```

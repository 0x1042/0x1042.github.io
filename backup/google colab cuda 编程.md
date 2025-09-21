# 环境搭建 

## 连接到GPU实例

> change runtime type -> GPU

<img width="1098" height="992" alt="Image" src="https://github.com/user-attachments/assets/d4ad2c85-d936-496d-bbe4-fcfef1c7ee2e" />

## 检查状态 

```python
!nvcc --version
!nvidia-smi
```

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2024 NVIDIA Corporation
Built on Thu_Jun__6_02:18:23_PDT_2024
Cuda compilation tools, release 12.5, V12.5.82
Build cuda_12.5.r12.5/compiler.34385749_0
Sun Sep 21 07:02:16 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla T4                       Off |   00000000:00:04.0 Off |                    0 |
| N/A   60C    P8             10W /   70W |       0MiB /  15360MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

## 安装插件 

```python
!pip install nvcc4jupyter
%load_ext nvcc4jupyter
```

## 修复版本 

from [learning-cuda-on-a-budget-on-google](https://leetarxiv.substack.com/p/learning-cuda-on-a-budget-on-google)

```
%%cuda -c "-I /does/not/exist -arch sm_75"
```

# hello world

```cuda
%%cuda -c "-I /does/not/exist -arch sm_75"
#include <stdio.h>

// This is a special function that runs on the GPU (device) instead of the CPU (host)
__global__ void kernel() {
  printf("Hello world!\n");
}

int main() {
  // Invoke the kernel function on the GPU with one block of one thread
  kernel<<<1,1>>>();

  // Check for error codes (remember to do this for _every_ CUDA function)
  if(cudaDeviceSynchronize() != cudaSuccess) {
    fprintf(stderr, "CUDA Error: %s\n", cudaGetErrorString(cudaPeekAtLastError()));
  }
  return 0;
}
```

<img width="2190" height="962" alt="Image" src="https://github.com/user-attachments/assets/7e5977be-514c-46df-a8fc-98587725b3d5" />


# 基本术语


## 函数修饰符

| 修饰符                | 含义                                    | 备注                                                     |
| --------------------- | --------------------------------------- | -------------------------------------------------------- |
| `__global__`          | 标记核函数                              | 在`host`(CPU) 端调用，由`device`（GPU）端执行            |
| `__device__`          | 标记设备函数                            | 只能在 GPU 上调用和执行，不能从 CPU 端直接调用           |
| `__host__`            | 标记主机函数，在CPU上执行（默认修饰符） | 只能在 CPU 上调用和执行（实际上 C++ 普通函数默认就host） |
| `__host__ __device__` |                                         | 可以同时在CPU和GPU上编译执行                             |

## 变量修饰符

| 修饰符         | 含义                                |
| -------------- | ----------------------------------- |
| `__shared__`   | 声明共享内存变量，`block`内线程共享 |
| `__constant__` | 声明常量内存变量，只读且有缓存      |
| `__device__`   | 声明全局设备变量                    |


## 执行模型

| 概念     | 含义                                                                         |
| -------- | ---------------------------------------------------------------------------- |
| `Kernel` | 在GPU上并行执行的函数，类似于CPU的函数但会被数千个线程同时执行               |
| `Grid`   | block 的集合，一个 kernel 启动时会指定一个 grid，grid 内包含多个 block       |
| `Block`  | 线程的集合，一个 kernel 的执行可以分为多个 block，每个 block 内有多个 thread |
| `Thread` | GPU 的最小执行单元，每个 kernel 启动后会并行运行许多线程                     |


## 内存层级 

## 内置变量 


| 概念              | 含义                    |
| ----------------- | ----------------------- |
| `threadIdx.x/y/z` | 当前线程在block中的索引 |
| `blockIdx.x/y/z`  | 当前block在grid中的索引 |
| `blockDim.x/y/z`  | block的维度大小         |
| `gridDim.x/y/z`   | grid的维度大小          |



# 语法

```
<<<Dg, Db, Ns, S>>>(args)
- Dg. Grid维度 
- Db. Block维度
- Ns. 动态共享内存大小
- S. Stream，用于异步执行
```
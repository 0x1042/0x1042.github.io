# 连接到GPU实例

> change runtime type -> GPU

<img width="1098" height="992" alt="Image" src="https://github.com/user-attachments/assets/d4ad2c85-d936-496d-bbe4-fcfef1c7ee2e" />

# 检查状态 

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

# 安装插件 

```python
!pip install nvcc4jupyter
%load_ext nvcc4jupyter
```

# 修复版本 

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



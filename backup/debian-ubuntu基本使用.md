# apt update

## warning

```
 sudo apt update
W: https://mirrors.tuna.tsinghua.edu.cn/llvm-apt/jammy/dists/llvm-toolchain-jammy-18/InRelease: 
Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg),
see the DEPRECATION section in apt-key(8) for details.
```


## 处理

- 找到对应的key 
```
sudo apt-key list

# 找到与warning相关的key
# 这里是llvm的 pub line 最后8个字符 

```

- 导入

```
sudo apt-key export AF4F7421 | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/llvm.gpg
```

- 重新update 

# 清理无效软连接

```shell

find . -xtype l

find /path -xtype l
```



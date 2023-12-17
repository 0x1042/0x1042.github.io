# 获取所有的环境变量 

- [获取所有的环境变量](#获取所有的环境变量)
  - [`main` 函数方式获取](#main-函数方式获取)
  - [`environ`读取](#environ读取)

## `main` 函数方式获取 

```cpp

auto main(int argc, char ** argv, char ** envp) -> int {
    logEnv(envp);
}

void logEnv(char ** envp) {
    LOG(INFO) << "current pid is " << getpid();
    for (char ** this_env = envp; *this_env != nullptr; ++this_env) {
        LOG(INFO) << "env:" << std::string(*this_env);
    }
}
```


## `environ`读取 

> 等同 `cat /proc/${PID}/environ | tr '\0' '\n'`

```cpp

auto main() -> int {
    logEnv2();
}

void logEnv2() {
    pid_t pid = getpid();
    std::clog << "current pid is " << pid << '\n';
    std::string envfile = "/proc/" + std::to_string(pid) + "/environ";

    std::ifstream input(envfile);

    std::string envkv;
    while (!input.eof()) {
        std::getline(input, envkv, '\0');
        std::clog << envkv << '\n';
    }

    input.close();
}
```

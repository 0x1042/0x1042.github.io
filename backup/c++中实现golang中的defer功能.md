# 使用`folly`

```c++
#include <folly/ScopeGuard.h>

auto main() -> int {
    std::string msg = "exit...";

    auto guard1 = folly::makeGuard([&] { std::cerr << "from folly::makeGuard:" << msg << '\n'; });

    std::cout << "Hello, World!" << '\n';
    return 0;
}

```

# 自定义实现

```c++
template <typename Lambda> 
struct Defer : Lambda {
    ~Defer() { Lambda::operator()(); }
};

template <typename Lambda> 
Defer(Lambda) -> Defer<Lambda>;

auto main() -> int {
    std::string msg = "exit...";

    Defer guard{[&] { std::cerr << msg << '\n'; }};

    std::cout << "Hello, World!" << '\n';
    return 0;
}
```

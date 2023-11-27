# 说明

> 仅限于读取类定义

```cpp

#include <memory>
#include <string>

#include <cxxabi.h>

template <typename T>
struct Name {
    Name() {
        std::unique_ptr<char, decltype(free) *> real_name(abi::__cxa_demangle(typeid(T).name(), nullptr, nullptr, nullptr), free);
        this->name = std::string(real_name.get());
    }

    std::string name{};
};

```

测试 

```cpp
namespace tests {

namespace a::b::c {
    class A {};
} // namespace a::b::c

namespace na {
    class B {};
} // namespace na

namespace a::b::d {
    struct C {};
} // namespace a::b::d

TEST_CASE("test name", "[name]") {
    // tests::a::b::c::A
    LOG(INFO) << "infra::Name<a::b::c::A>().name:" << infra::Name<a::b::c::A>().name;
    // tests::na::B
    LOG(INFO) << "infra::Name<na::B>().name:" << infra::Name<na::B>().name;
    // tests::a::b::d::C
    LOG(INFO) << "infra::Name<a::b::d::C>().name:" << infra::Name<a::b::d::C>().name;
}
} // namespace tests
```

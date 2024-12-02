# 背景

> 上线了业务逻辑代码，发现有`coredump`, 从堆栈看，是出现了越界。通过缩小范围，最小化复现的代码如下

```c++
    std::random_device rd;
    std::default_random_engine rng(rd());
    std::uniform_int_distribution<int> dis(0, 1);

    std::sort(items.begin(), items.end(), [&dis, &rng](const Item & lhs, const Item & rhs) {
        if (std::fabs(lhs.score - rhs.score) > EPSILON) {
            return lhs.score > rhs.score;
        }
        return dis(rng) % 2 == 0;
    });
```

# 问题 

c++要求sort的cmp函数要满足严格弱序，否则会引发越界或者为定义行为 

https://en.cppreference.com/w/cpp/named_req/Compare
 



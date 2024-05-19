# parallel

## fetch 

```c++
auto fetch(const std::string & url) -> int {
    Timer start;
    cpr::Response r = cpr::Get(cpr::Url{url}, cpr::Timeout{std::chrono::milliseconds(200)});
    LOG(INFO) << "thread-" << std::this_thread::get_id() << " fetch [" << url << "] cost:" << start.get_elapsed_ms()
              << " ms. status_code: " << r.status_code << " text:" << r.text.size() << '\n';
    return 0;
}
```

## folly
 
```c++
    Timer start;

    std::vector<std::string> urls = {
        "https://www.cnblogs.com/",
        "https://www.alipan.com/",
        "https://www.toutiao.com/",
    };

    std::vector<folly::Future<int>> futures;
    futures.reserve(urls.size());

    for (const auto & url : urls) {
        auto fn = folly::via(infra::global().get(), [url]() { return infra::fetch(url); });
        futures.push_back(std::move(fn));
    }

    folly::collectAll(futures).get();

    LOG(INFO) << "folly fetch " << urls.size() << " total cost:" << start.get_elapsed_ms() << " ms\n";
```

## omp 

```c++
    Timer start;

    std::vector<std::string> urls = {
        "https://www.cnblogs.com/",
        "https://www.alipan.com/",
        "https://www.toutiao.com/",
    };

#pragma omp parallel for num_threads(urls.size())
    for (size_t i = 0; i < urls.size(); i++) {
        int status = infra::fetch(urls[i]);
        LOG(INFO) << "status:" << status;
    }
    LOG(INFO) << "omp fetch " << urls.size() << " total cost:" << start.get_elapsed_ms() << " ms\n";
```
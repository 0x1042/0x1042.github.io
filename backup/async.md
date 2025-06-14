# 异步

## 线程

同步的实现

```C++
auto get(const std::string &url) -> std::string {
  ScopedTimer timer(__PRETTY_FUNCTION__);
  cpr::Response rsp = cpr::Get(cpr::Url{url});
  return rsp.text;
}
```

### `std::async`

```C++
auto get_async(const std::string &url) -> std::future<std::string> {
  return std::async(std::launch::async, get, url);
}

TEST(get, get2) {
  const std::string url = "https://www.bilibili.com";
  auto &&rsp = get_async(url);
  LOG(INFO) << "get async start ";
  const auto &content = rsp.get();
  LOG(INFO) << "content is " << content;
}
```

### `std::packaged_task`

> [!Tip]
> <ins>**1. packaged_task 参数是一个无参的可调用对象**</ins>
> <ins>**2. 在任务被执行前，先获取它的 future**</ins>
> <ins>**3. 将任务提交到线程池**</ins>

```c++
auto get_async2(const std::string &url) -> std::future<std::string> {
  auto fn = [url]() { return get(url); };

  std::packaged_task<std::string()> task(std::move(fn));
  std::future<std::string> future = task.get_future();
  thread_pool()->add([t = std::move(task)]() mutable { t(); });

  return future;
}
```

### `folly::via`

```C++
auto get_async3(const std::string &url) -> folly::Future<std::string> {
  return folly::via(thread_pool().get(), [url]() { return get(url); });
}
```

## 协程

> todo
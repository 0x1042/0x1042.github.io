# 解析域名

```c++
    std::string domain = "www.toutiao.com";
    std::string port = "443";

    asio::ip::tcp::resolver resolver(socket.get_executor());
    asio::ip::tcp::resolver::results_type endpoints =
        co_await resolver.async_resolve(host, port, asio::use_awaitable);
```

# tcp连接 

```c++

// 连接解析后的结果 

asio::ip::tcp::socket remote_socket(socket.get_executor());
co_await asio::async_connect(remote_socket, endpoints, asio::use_awaitable);

// 连接host port
asio::ip::tcp::endpoint remote_endpoint(asio::ip::make_address_v4(address),
                                            (port[0] << 8) | port[1]);
asio::ip::tcp::socket remote_socket(socket.get_executor());
co_await remote_socket.async_connect(remote_endpoint, asio::use_awaitable);
```

# copy 两个socket 

```c++
using namespace asio::experimental::awaitable_operators;

constexpr size_t bufsize = 16 * 1024;

asio::awaitable<void> relay(asio::ip::tcp::socket &from,
                            asio::ip::tcp::socket &to) {

  auto relay = [](asio::ip::tcp::socket &from,
                  asio::ip::tcp::socket &to) -> asio::awaitable<void> {
    try {
      std::array<uint8_t, bufsize> data{};
      for (;;) {
        std::size_t n = co_await from.async_read_some(asio::buffer(data),
                                                      asio::use_awaitable);
        co_await asio::async_write(to, asio::buffer(data, n),
                                   asio::use_awaitable);
      }
    } catch (...) {
      from.close();
      to.close();
    }
  };

  co_await (relay(from, to) && relay(to, from));
}


// 使用 
co_await relay(socket, remote_socket);

```

# 读取指定长度数据 

```c++
std::array<uint8_t, 4> handshake_request{};
co_await asio::async_read(socket, asio::buffer(handshake_request), asio::use_awaitable);
```

# 按行读取 

```c++
    asio::streambuf buffer;
    std::vector<std::string> lines;
    for (;;) {
      std::size_t n = co_await asio::async_read_until(socket, buffer, "\r\n",
                                                      asio::use_awaitable);
      auto bufs = buffer.data();
      std::string line(asio::buffers_begin(bufs),
                       asio::buffers_begin(bufs) + n);

      if (line == "\r\n") {
        break;
      }

      lines.push_back(std::move(line));
      buffer.consume(n);
    }
```

# peek 

```c++
std::array<uint8_t, 1> data{};
socket.receive(asio::buffer(data), asio::socket_base::message_peek);
std::clog << "message_peek:" << fmt::to_string(data) << '\n';
```


# 写入数据

- 写入string

```c++
const std::string &response = "HTTP/1.1 200 Connection Established\r\n\r\n";
co_await asio::async_write(socket, asio::buffer(response),
                               asio::use_awaitable);
```

- 写入bytes

```c++
std::array<uint8_t, 10> response = {
        0x05,       0x00,       0x00,       0x01,    address[0],
        address[1], address[2], address[3], port[0], port[1]};
co_await asio::async_write(socket, asio::buffer(response),
                               asio::use_awaitable);
```

# listen

```c++
auto endpoint = asio::ip::tcp::endpoint(asio::ip::tcp::v4(), port);
asio::ip::tcp::acceptor acceptor(io_context, endpoint);

  for (;;) {
    asio::ip::tcp::socket socket = co_await acceptor.async_accept(asio::use_awaitable);
  }

```




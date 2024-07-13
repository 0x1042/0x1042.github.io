# listen

```c++
asio::awaitable<void> listener(asio::io_context &io_context,
                               unsigned short port) {
  auto endpoint = asio::ip::tcp::endpoint(asio::ip::tcp::v4(), port);
  spdlog::info("server listen at {}:{}", endpoint.address().to_string(),
               endpoint.port());

  asio::ip::tcp::acceptor acceptor(io_context, endpoint);
  for (;;) {
    asio::ip::tcp::socket socket =
        co_await acceptor.async_accept(asio::use_awaitable);

    std::array<std::byte, 1> data{};
    socket.receive(asio::buffer(data), asio::socket_base::message_peek);
    const auto &endpoint = socket.remote_endpoint();
    spdlog::info("incoming request. {}:{}", endpoint.address().to_string(),
                 endpoint.port());

    if (std::to_integer<uint8_t>(data.at(0)) == 0x05) {
      asio::co_spawn(io_context, socks::handle(std::move(socket)),
                     asio::detached);
    } else {
      asio::co_spawn(io_context, http::handle(std::move(socket)),
                     asio::detached);
    }
  }
```

# http

```c++
asio::awaitable<void> handle(asio::ip::tcp::socket socket) {
  try {
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

    std::string line = replace_all(lines.at(1), "\r\n", "");
    line = replace_all(line, "Host: ", "");

    std::string port = "80";

    std::vector<std::string> hostport = absl::StrSplit(line, ":");
    std::string host = hostport[0];
    if (hostport.size() == 2) {
      port = hostport[1];
    }

    asio::ip::tcp::resolver resolver(socket.get_executor());
    asio::ip::tcp::resolver::results_type endpoints =
        co_await resolver.async_resolve(host, port, asio::use_awaitable);

    asio::ip::tcp::socket remote_socket(socket.get_executor());

    // todo http

    co_await asio::async_connect(remote_socket, endpoints, asio::use_awaitable);

    spdlog::info("connect to {}:{} success. ",
                 endpoints->endpoint().address().to_string(),
                 endpoints->endpoint().port());

    const std::string &response = "HTTP/1.1 200 Connection Established\r\n\r\n";
    co_await asio::async_write(socket, asio::buffer(response),
                               asio::use_awaitable);
    co_await relay(socket, remote_socket);

  } catch (const std::exception &ex) {
    spdlog::error("run exception :{}", ex.what());
  }
}
```

# socks5

```c++
asio::awaitable<void> handle(asio::ip::tcp::socket socket) {
  try {
    // Perform SOCKS5 handshake
    std::array<uint8_t, 4> handshake_request{};
    co_await asio::async_read(socket, asio::buffer(handshake_request),
                              asio::use_awaitable);

    if (handshake_request[0] != 0x05) {
      co_return; // Not SOCKS5
    }
    std::array<uint8_t, 2> handshake_response = {0x05, 0x00};
    co_await asio::async_write(socket, asio::buffer(handshake_response),
                               asio::use_awaitable);

    // Read SOCKS5 request
    std::array<uint8_t, 4> request{};
    co_await asio::async_read(socket, asio::buffer(request),
                              asio::use_awaitable);

    if (request[1] != 0x01) {
      co_return; // Only support CONNECT command
    }

    // Read address and port
    std::array<uint8_t, 4> address{};
    co_await asio::async_read(socket, asio::buffer(address),
                              asio::use_awaitable);

    std::array<uint8_t, 2> port{};
    co_await asio::async_read(socket, asio::buffer(port), asio::use_awaitable);

    asio::ip::tcp::endpoint remote_endpoint(asio::ip::make_address_v4(address),
                                            (port[0] << 8) | port[1]);

    spdlog::info("connect to {}:{} success. ",
                 remote_endpoint.address().to_string(), remote_endpoint.port());

    asio::ip::tcp::socket remote_socket(socket.get_executor());

    // Connect to the remote server
    co_await remote_socket.async_connect(remote_endpoint, asio::use_awaitable);

    // Send success response to the client
    std::array<uint8_t, 10> response = {
        0x05,       0x00,       0x00,       0x01,    address[0],
        address[1], address[2], address[3], port[0], port[1]};
    co_await asio::async_write(socket, asio::buffer(response),
                               asio::use_awaitable);

    // Relay traffic between client and remote server
    co_await relay(socket, remote_socket);
  } catch (const std::exception &ex) {
    spdlog::error("run exception :{}", ex.what());
  }
}
```

# relay

```c++
asio::awaitable<void> relay(asio::ip::tcp::socket &from,
                            asio::ip::tcp::socket &to) {

  auto relay = [](asio::ip::tcp::socket &from,
                  asio::ip::tcp::socket &to) -> asio::awaitable<void> {
    const auto &from_addr = from.remote_endpoint();
    const auto &to_addr = to.remote_endpoint();
    size_t cnt = 0;
    try {
      std::array<std::byte, bufsize> data{};
      for (;;) {
        std::size_t n = co_await from.async_read_some(asio::buffer(data),
                                                      asio::use_awaitable);
        co_await asio::async_write(to, asio::buffer(data, n),
                                   asio::use_awaitable);
        cnt += n;
      }
    } catch (...) {
      from.close();
      to.close();
    }

    spdlog::info("{}:{} -> {}:{} transfer {} bytes success. ",
                 from_addr.address().to_string(), from_addr.port(),
                 to_addr.address().to_string(), to_addr.port(), cnt);
  };

  co_await (relay(from, to) && relay(to, from));
}
```

# main

```c++
ABSL_FLAG(uint16_t, port, 10010, "server listen port");
ABSL_FLAG(size_t, worker, 4, "worker num");

int main(int argc, char **argv) {

  absl::ParseCommandLine(argc, argv);

  auto console = spdlog::stdout_color_mt("console");
  spdlog::set_default_logger(console);
  spdlog::set_pattern("%^[%H:%M:%S %e] %l thread-%t %v %$");

  try {
    asio::io_context io_context;

    // Run listener coroutine
    asio::co_spawn(io_context, listener(io_context, absl::GetFlag(FLAGS_port)),
                   asio::detached);

    std::vector<std::thread> threads;
    size_t worker_num = absl::GetFlag(FLAGS_worker);

    threads.resize(worker_num);
    for (size_t i = 0; i < worker_num; ++i) {
      threads.emplace_back([&io_context]() { io_context.run(); });
    }

    io_context.run();

    // Join all threads
    for (auto &t : threads) {
      t.join();
    }
  } catch (const std::exception &ex) {
    spdlog::error("run exception :{}", ex.what());
  }
  return 0;
}
```

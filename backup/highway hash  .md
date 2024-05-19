# highway hash 

- c [https://github.com/google/highwayhash](https://github.com/google/highwayhash) 
- go [https://github.com/minio/highwayhash](https://github.com/minio/highwayhash)

# go 

## 64bit 

```go

var (
	key [32]byte
)

func HighHash(input string) string {
	hash, _ := highwayhash.New64(key[:])
	hash.Write([]byte(input))
	checksum := hash.Sum(nil)
	return hex.EncodeToString(checksum)
}

func HighHashFile(path string) string {
	hash, _ := highwayhash.New64(key[:])
	file, _ := os.Open(path)
	io.Copy(hash, file)
	return hex.EncodeToString(hash.Sum(nil))
}
```


## 128 bit

```go
func HighHash128(input string) string {
	hash, _ := highwayhash.New128(key[:])
	hash.Write([]byte(input))
	checksum := hash.Sum(nil)
	return hex.EncodeToString(checksum)
}
```

# c++

## 64bit 

```c++

void put_uint64_le(uint8_t *buf, uint64_t value) {
  buf[0] = static_cast<uint8_t>(value & 0xFF);
  buf[1] = static_cast<uint8_t>((value >> 8) & 0xFF);
  buf[2] = static_cast<uint8_t>((value >> 16) & 0xFF);
  buf[3] = static_cast<uint8_t>((value >> 24) & 0xFF);
  buf[4] = static_cast<uint8_t>((value >> 32) & 0xFF);
  buf[5] = static_cast<uint8_t>((value >> 40) & 0xFF);
  buf[6] = static_cast<uint8_t>((value >> 48) & 0xFF);
  buf[7] = static_cast<uint8_t>((value >> 56) & 0xFF);
}

const static uint64_t EMPTY_KEY[4] = {0, 0, 0, 0};

std::string high_hash(const std::string &input) {
  uint64_t hash = HighwayHash64(reinterpret_cast<const uint8_t *>(input.data()),
                                input.size(), EMPTY_KEY);
  uint8_t bytes[8];
  put_uint64_le(bytes, hash);
  std::string rsp;
  for (int i = 0; i < 8; i++) {
    rsp += fmt::format("{0:02x}", bytes[i]);
  }
  return rsp;
}
```

## 128 bit

```c++

std::string high_hash128(const std::string &input) {
  uint64_t hash[2];
  HighwayHash128(reinterpret_cast<const uint8_t *>(input.data()), input.size(),
                 EMPTY_KEY, hash);
  uint8_t bytes1[8];
  put_uint64_le(bytes1, hash[0]);

  uint8_t bytes2[8];
  put_uint64_le(bytes2, hash[1]);
  std::string rsp;
  for (int i = 0; i < 8; i++) {
    rsp += fmt::format("{0:02x}", bytes1[i]);
  }
  for (int i = 0; i < 8; i++) {
    rsp += fmt::format("{0:02x}", bytes2[i]);
  }
  return rsp;
}
```

## 优化版本

```c++
auto high_hash128_v2(const std::string & input) -> std::string {
    uint64_t hash[2];
    HighwayHash128(reinterpret_cast<const uint8_t *>(input.data()), input.size(), EMPTY_KEY, hash);

    std::array<uint8_t, 16> buffer{};
    std::memcpy(buffer.data(), &hash, sizeof(hash));

    std::string rsp;
    for (unsigned char & b : buffer) {
        rsp += fmt::format("{0:02x}", b);
    }
    return rsp;
}
```

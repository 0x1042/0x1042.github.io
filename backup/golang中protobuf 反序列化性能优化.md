# 背景

线上有一些公共使用的超大`protobuf`，部分单**message的field大于10000**。由于业务侧只使用部分字段，所以针对这个场景做裁剪，只保留使用到的字段。但是从火焰图上看，反序列化的开销仍然很大。

从`protobuf`[官方文档](https://protobuf.dev/programming-guides/proto3/#unknowns)看，反序列化时，不能识别的字段会放在`unknown fields `，**proto3** 版本一开始时丢弃`unknown fileds`，**但是从3.5 开始对齐proto2的功能，默认保留**

<img width="1367" alt="image" src="https://github.com/0x1042/0x1042.github.io/assets/7525242/9d3d3f4d-ff58-4b6b-a65c-ddd6eabbba58">

# `DiscardUnknown`

从官方的api看，新版本的api支持在unmarshal时，指定`UnmarshalOptions`，其中一个开关就是 `DiscardUnknown`, 使用方式如下 

```go
import (
	"google.golang.org/protobuf/proto"
)

// BaseParser  线上基线版本
func BaseParser[T proto.Message](buf []byte, t T) error {
	return proto.Unmarshal(buf, t)
}

// AbtestParser  新版本
func AbtestParser[T proto.Message](buf []byte, t T) error {
	opt := proto.UnmarshalOptions{
		DiscardUnknown: true,
	}
	return opt.Unmarshal(buf, t)
}

```

# 性能差异

抓取一个场景的数据，单个message 序列化之后在30kb左右，从benchmark看，可以提升34%的性能，内存分配下降97%。实际效果取决于裁剪字段站比，需要实际场景测试验证，不具有参考价值。

```go

func BenchmarkBaseParser(b *testing.B) {
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		target := simple.BidRequest{}

		if err := BaseParser(buf, &target); err != nil {
			panic(err)
		}
	}
}

func BenchmarkAbtestParser(b *testing.B) {
	b.ResetTimer()

	for i := 0; i < b.N; i++ {
		target := simple.BidRequest{}

		if err := AbtestParser(buf, &target); err != nil {
			panic(err)
		}
	}
}

```

> benchmark

```shell
go test -bench=. -count=2  -benchtime=5s -benchmem -cpuprofile=cpu.out -memprofile=mem.out
```

> 对比

```shell

# 先使用base 
go test -bench=. -count=10  -benchtime=5s -benchmem > old.txt
# 使用abtest 
go test -bench=. -count=10  -benchtime=5s -benchmem > new.txt

benchstat old.txt new.txt

```

# cpp

> cpp [也有类似的api](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/#Message.DiscardUnknownFields.details)，但是实在 parse之后执行，对于反序列化性能没有帮助，但是对于缓存场景，可以降低内存的使用率

```cpp
void string_to_message(const std::string & src, tutorial::Person & dst) {
    if (bool status = dst.ParseFromString(src); !status) {
        LOG(ERROR) << "unmarshal fail";
    }
    dst.DiscardUnknownFields();
}
``` 
<img width="1336" alt="image" src="https://github.com/0x1042/0x1042.github.io/assets/7525242/dd86b27b-bc50-412d-981c-e3fcebd0d605">



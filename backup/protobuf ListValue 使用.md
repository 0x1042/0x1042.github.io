# 背景

> 考虑这个`json` 如何定义对应的`protobuf` 结构？

```json
{
    "show_list": {
        "hello1": [
            "world1",
            "world1"
        ],
        "hello2": [
            "world2",
            "world2"
        ]
    }
}
```

**`protobuf` 中无法直接声明`map<key,repeated values>` 对应的结构，需要借助`ListValue`**

# 定义

```protobuf
syntax = "proto3";

package demo;
option go_package = "./demos";

import "google/protobuf/struct.proto";

message UaInfo {
  map<string, google.protobuf.ListValue> show_list = 1;
}

```

# cpp 

> 生成 `protoc --proto_path=proto --cpp_out=pbgen proto/*.proto`

```cpp
TEST(pb, valus) {
    const std::string json = R"({"show_list":{"hello1":["world1","world1"],"hello2":["world2","world2"]}})";

    google::protobuf::json::ParseOptions opt;
    opt.ignore_unknown_fields = true;
    opt.case_insensitive_enum_parsing = true;
    demo::UaInfo info;

    const auto & status = google::protobuf::json::JsonStringToMessage(json, &info, opt);

    std::clog << "status " << status << '\n';

    std::clog << info.DebugString() << '\n';

    std::unordered_map<std::string, std::vector<std::string>> db;

    for (const auto & inner : info.show_list()) {
        for (const auto & list : inner.second.values()) {
            db[inner.first].push_back(list.string_value());
        }
    }

    // 输出 {"hello1": ["world1", "world1"], "hello2": ["world2", "world2"]}
    std::clog << fmt::to_string(db) << '\n';
}
```

# go

> 生成 `protoc --proto_path=. --go_out=. --go_opt=paths=source_relative *.proto`

- 反序列化

```go
func Test_test(t *testing.T) {
	str := "{\"show_list\":{\"hello1\":[\"world1\",\"world1\"],\"hello2\":[\"world2\",\"world2\"]}}"
	uainfo := new(UaInfo)

	if err := json.Unmarshal([]byte(str), uainfo); err != nil {
		log.Fatal(err)
	}

	log.Printf("%+v", uainfo)

	bs, err := json.Marshal(uainfo)

	if err != nil {
		panic(err)
	}
	log.Printf("\n%s", bs)
}

```

- 生成

```go
func Test_test2(t *testing.T) {
	uainfo := new(UaInfo)
	uainfo.ShowList = make(map[string]*structpb.ListValue)
	l1, _ := structpb.NewList([]any{"world101", "world102"})
	uainfo.ShowList["hello1"] = l1

	l2, _ := structpb.NewList([]any{"world202", "world201"})
	uainfo.ShowList["hello2"] = l2

	bs, err := json.Marshal(uainfo)

	if err != nil {
		panic(err)
	}
	log.Printf("uainfo\n%s", bs)
}
```

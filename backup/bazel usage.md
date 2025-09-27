# bazel samples

## `use protobuf`

- `MODULE.bazel`

```bazel{highlight=1 .line-numbers}
bazel_dep(name = "protobuf", version = "32.1")
bazel_dep(name = "rules_proto", version = "7.1.0")
```

- `app/BUILD.bazel`

```bazel{highlight=1 .line-numbers}
load("@protobuf//bazel:cc_proto_library.bzl", "cc_proto_library")
load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library")
load("@rules_proto//proto:defs.bzl", "proto_library")

proto_library(
    name = "graph_proto",
    srcs = [
        "idls/graph.proto",
    ],
    visibility = ["//visibility:public"],
)

cc_proto_library(
    name = "graph_cc_proto",
    visibility = ["//visibility:public"],
    deps = [":graph_proto"],
)
```

## `replace malloc`

```starlark{highlight=1 .line-numbers}
bazel_dep(name = "mimalloc", version = "2.2.4")

cc_binary(
    name = "unittest2",
    srcs = [
        "main.cc",
    ],
    copts = DEFAULT_COPTS,
    linkopts = DEFAULT_LINKOPTS,
    malloc = "@mimalloc",
    deps = [
        "@mimalloc//:mimalloc-api"
    ],
)
```

```CPP{highlight=1 .line-numbers}
#include <mimalloc.h>

auto main(int argc, char ** argv) -> int {
    LOG(INFO) << "mimalloc version " << mi_version();
    return RUN_ALL_TESTS();
}
```

## `asan`

```
# .bazelrc
# Address Sanitizer
build:asan --strip=never
build:asan --copt=-fsanitize=address
build:asan --copt=-fno-omit-frame-pointer
build:asan --copt=-fno-optimize-sibling-calls
build:asan --copt=-g
build:asan --copt=-O0
build:asan --linkopt=-fsanitize=address

# Undefined Behavior Sanitizer
build:ubsan --copt=-fsanitize=undefined
build:ubsan --copt=-fsanitize-trap=undefined
build:ubsan --copt=-O0
build:ubsan --copt=-fno-omit-frame-pointer
build:ubsan --copt=-fno-optimize-sibling-calls
build:ubsan --copt=-g
build:ubsan --linkopt=-fsanitize=undefined

# Thread Sanitizer
build:tsan --copt=-fsanitize=thread
build:tsan --copt=-fno-omit-frame-pointer
build:tsan --copt=-fno-optimize-sibling-calls
build:tsan --copt=-g
build:tsan --copt=-O1
build:tsan --linkopt=-fsanitize=thread

# Leak Sanitizer
build:lsan --copt=-fsanitize=leak
build:lsan --copt=-fno-omit-frame-pointer
build:lsan --copt=-fno-optimize-sibling-calls
build:lsan --copt=-g
build:lsan --linkopt=-fsanitize=leak

# Memory Sanitizer (仅 clang)
build:msan --copt=-fsanitize=memory
build:msan --copt=-fno-omit-frame-pointer
build:msan --copt=-fno-optimize-sibling-calls
build:msan --copt=-g
build:msan --copt=-O1
build:msan --linkopt=-fsanitize=memory
```


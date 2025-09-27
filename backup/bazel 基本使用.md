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

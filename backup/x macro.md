# x 宏

> 实现枚举与str互相转换的时候，搜到的一个用法。
> 核心是 使用替换x宏的实现，来使用同一份数据生成不同的定义，好处是减少冗余实现和防止不一致

## step 1 定义数据类型别表

```c
#define HTTP_METHODS                                                           \
  X(HTTP_GET, "GET")                                                           \
  X(HTTP_POST, "POST")                                                         \
  X(HTTP_HEAD, "HEAD")                                                         \
  X(HTTP_PUT, "PUT")                                                           \
  X(HTTP_CONNECT, "CONNECT")                                                   \
  X(HTTP_TRACE, "TRACE")                                                       \
  X(HTTP_DELETE, "DELETE")                                                     \
  X(HTTP_OPTIONS, "OPTIONS")
```

## step2 定义枚举类型

```c
#define X(val, desc) val,

typedef enum { HTTP_METHODS HTTP_UNKNOWN } http_method_t;

#undef X
```

这里的`X`宏是将 `step 1`的类型列表中的枚举提取出来，生成`enum` 的定义

宏解析结果

```c
typedef enum {
  HTTP_GET,
  HTTP_POST,
  HTTP_HEAD,
  HTTP_PUT,
  HTTP_CONNECT,
  HTTP_TRACE,
  HTTP_DELETE,
  HTTP_OPTIONS,
  HTTP_UNKNOWN
} http_method_t;
```

## step3 定义使用字符串数组 

```c
#define X(val, desc) desc,

static const char *http_method_infos[] = {HTTP_METHODS};

#undef X
```

宏解析结果

```c
static const char *http_method_infos[] = {
    "GET", "POST", "HEAD", "PUT", "CONNECT", "TRACE", "DELETE", "OPTIONS",
};
```

## step4 使用

```c
const char *mhd_to_str(http_method_t mhd);
http_method_t str_to_mhd(const char *str);

const char *mhd_to_str(http_method_t mhd) {
  if (mhd >= 0 && mhd < HTTP_UNKNOWN) {
    return http_method_infos[mhd];
  }

  return "UNKNOWN";
}

http_method_t str_to_mhd(const char *str) {
  for (int i = 0; i < HTTP_UNKNOWN; i++) {
    if (strcasecmp(str, http_method_infos[i]) == 0) {
      return (http_method_t)i;
    }
  }
  return HTTP_UNKNOWN;
}

```




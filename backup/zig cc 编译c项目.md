# zig cc 编译c项目 

## 单个c文件

```zig
    const flags = .{
        "-Wall",
        "-Wextra",
        "-Wpedantic",
        "-Werror=return-type",
        "-std=gnu23",
    };

    exe.addCSourceFile(.{
        .file = std.Build.LazyPath{ .cwd_relative = "http.c" },
        .flags = &flags,
    });

    exe.linkLibC();
```

## 多个c文件

```zig
    const cfiles = .{
        "http.c",
        "log.c",
        "server.c",
        "main.c",
        "relay.c",
        "socks.c",
    };

    exe.addCSourceFiles(.{
        .files = &cfiles,
        .flags = &flags,
    });
```

## 自定义目录和后缀

```zig

    {
        var sources = std.ArrayList([]const u8).init(b.allocator);
        var dir = try std.fs.cwd().openDir(".", .{ .access_sub_paths = true });
        var walker = try dir.walk(b.allocator);
        defer walker.deinit();

        const allowed_exts = [_][]const u8{ ".c", ".cpp", ".cxx", ".c++", ".cc" };

        while (try walker.next()) |entry| {
            const ext = std.fs.path.extension(entry.basename);
            const include_file = for (allowed_exts) |e| {
                if (std.mem.eql(u8, ext, e))
                    break true;
            } else false;
            if (include_file) {
                // we have to clone the path as walker.next() or walker.deinit() will override/kill it
                try sources.append(b.dupe(entry.path));
            }
        }
    }
```

## todo1 外部库 

## todo2 c++


## 编译 

- native 

```zig 

VERBOSE := "--verbose --summary all"
zb_rel:
    zig build {{ VERBOSE }} --release=safe -Doptimize=ReleaseSafe --prefix {{ OUTPUT }}/rel

```
- 交叉

```zig 
zb_linux_rel:
    zig build {{ VERBOSE }} -Dtarget=x86_64-linux-gnu -Doptimize=ReleaseSafe --prefix {{ OUTPUT }}/linux/rel
```


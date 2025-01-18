# 跨平台编译

```zig

const targets: []const std.Target.Query = &.{
    .{ .cpu_arch = .x86_64, .os_tag = .macos },
    .{ .cpu_arch = .x86_64, .os_tag = .linux, .abi = .gnu },
    .{ .cpu_arch = .x86_64, .os_tag = .linux, .abi = .musl },
    .{ .cpu_arch = .x86_64, .os_tag = .windows },
    .{ .cpu_arch = .aarch64, .os_tag = .macos },
    .{ .cpu_arch = .aarch64, .os_tag = .linux },
    .{ .cpu_arch = .aarch64, .os_tag = .windows },
    .{ .cpu_arch = .mips, .os_tag = .linux, .abi = .musl },
    .{ .cpu_arch = .mipsel, .os_tag = .linux, .abi = .musl },
    .{ .cpu_arch = .mips64, .os_tag = .linux, .abi = .musl },
    .{ .cpu_arch = .mips64el, .os_tag = .linux, .abi = .musl },
};

pub fn build(b: *std.Build) void {
    const optimize = b.standardOptimizeOption(.{});

    for (targets) |t| {
        var name: []u8 = undefined;
        const arch = if (t.cpu_arch) |arch| @tagName(arch) else "native";
        const os = if (t.os_tag) |os| @tagName(os) else "unknown";
        if (t.abi) |abi| {
            name = b.fmt("{s}-{s}-{s}", .{ os, arch, @tagName(abi) });
        } else {
            name = b.fmt("{s}-{s}", .{ os, arch });
        }

        const exe = b.addExecutable(.{
            .name = name,
            .root_source_file = b.path("src/main.zig"),
            .target = b.resolveTargetQuery(t),
            .optimize = optimize,
        });

        b.installArtifact(exe);
    }
}

```

# 自定义编译选项 


```zig

fn addOption(b: *std.Build) *std.Build.Step.Options {
    const version = b.option([]const u8, "version", "build version") orelse "dev";
    const date = b.option([]const u8, "date", "build date") orelse "unknown";

    const options = b.addOptions();
    options.addOption([]const u8, "version", version);
    options.addOption([]const u8, "date", date);

    return options;
}

exe.root_module.addOptions("config", options);

```

编译时，指定选项

```shell
zig build -freference-trace --summary all --verbose -Dversion=22c0548 -Ddate=2025-01-18
```

`addOptions` 会将选项转换成zig代码，同时可以在项目中直接依赖，可以使用这个功能来生成与`golang`中通过 `ldflags -X` 编译时生成构建日期/commit等一样的功能

```zig
// main.zig

const config = @import("config");

const version = config.date ++ " " ++ config.version;

```

# 根据编译类型控制日志级别

```zig
pub const level = switch (@import("builtin").mode) {
    .Debug => std.log.Level.debug,
    else => std.log.Level.info,
};

pub const std_options = .{
    .log_level = level,
    .logFn = logger.logfn,
};
```




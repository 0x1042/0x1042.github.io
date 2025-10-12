# 配置

`CMakePresets.json
`
```json
{
    "configurePresets": [
        {
            "name": "base",
            "hidden": true,
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/build/${presetName}",
            "cacheVariables": {
                "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake",
                "CMAKE_C_STANDARD": "23",
                "CMAKE_C_STANDARD_REQUIRED": "ON",
                "CMAKE_C_EXTENSIONS": "OFF",
                "CMAKE_CXX_STANDARD": "20",
                "CMAKE_CXX_EXTENSIONS": "OFF",
                "CMAKE_CXX_STANDARD_REQUIRED": "ON",
                "CMAKE_EXPORT_COMPILE_COMMANDS": "ON",
                "CMAKE_COLOR_DIAGNOSTICS": "ON"
            },
            "environment": {
                "CC": "/usr/bin/clang",
                "CXX": "/usr/bin/clang++"
            }
        },
        {
            "name": "debug",
            "inherits": "base",
            "displayName": "Debug Build",
            "description": "Debug configuration with symbols",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug"
            }
        },
        {
            "name": "release",
            "inherits": "base",
            "displayName": "Release Build",
            "description": "Optimized release without debug symbols",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release",
                "CMAKE_INTERPROCEDURAL_OPTIMIZATION": "ON",
                "CMAKE_CXX_FLAGS_RELEASE": "-O3 -DNDEBUG"
            }
        },
        {
            "name": "releasewithdbg",
            "inherits": "base",
            "displayName": "Release with Debug Info",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "RelWithDebInfo",
                "CMAKE_CXX_FLAGS_RELWITHDEBINFO": "-O3 -g",
                "CMAKE_INTERPROCEDURAL_OPTIMIZATION": "ON",
                "CMAKE_POSITION_INDEPENDENT_CODE": "ON"
            }
        },
        {
            "name": "asan",
            "inherits": "debug",
            "displayName": "Address Sanitizer",
            "description": "Detect memory errors (ASan + UBSan)",
            "environment": {
                "SANITIZE_FLAGS": "-fsanitize=address,undefined -fno-omit-frame-pointer -fno-optimize-sibling-calls",
                "ASAN_OPTIONS": "abort_on_error=1:disable_coredump=0:detect_leaks=1"
            },
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_C_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_CXX_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_EXE_LINKER_FLAGS": "$env{SANITIZE_FLAGS}"
            }
        },
        {
            "name": "tsan",
            "inherits": "debug",
            "displayName": "Thread Sanitizer",
            "cacheVariables": {
                "CMAKE_C_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_CXX_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_EXE_LINKER_FLAGS": "$env{SANITIZE_FLAGS}"
            },
            "environment": {
                "SANITIZE_FLAGS": "-fsanitize=thread -fno-omit-frame-pointer",
                "TSAN_OPTIONS": "halt_on_error=1"
            }
        },
        {
            "name": "ubsan",
            "inherits": "debug",
            "displayName": "Undefined Behavior Sanitizer",
            "description": "Detect integer overflow, invalid shifts, etc.",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_C_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_CXX_FLAGS": "$env{SANITIZE_FLAGS}",
                "CMAKE_EXE_LINKER_FLAGS": "$env{SANITIZE_FLAGS}"
            },
            "environment": {
                "SANITIZE_FLAGS": "-fsanitize=undefined -fno-omit-frame-pointer",
                "UBSAN_OPTIONS": "halt_on_error=1"
            }
        }
    ],
    "cmakeMinimumRequired": {
        "major": 4,
        "minor": 0,
        "patch": 0
    },
    "version": 3
}
```

# usage

```makefile
PRESET ?= asan

RED=\033[31m
GREEN=\033[32m
YELLOW=\033[33m
RESET=\033[0m

START := $(shell date +%s)

.PHONY: all config build clean info

all: build
	@END=$$(date +%s); \
	ELAPSED=$$((END - $(START))); \
	printf "$(GREEN)%s  build time cost: %s seconds$(RESET)\n" "$$(date '+%Y-%m-%d %H:%M:%S')" "$${ELAPSED}"

info:
	cmake --list-presets
	@printf "\n"

config: info
	@printf "$(YELLOW)%s Configuring with preset: %s$(RESET)\n" "$$(date '+%Y-%m-%d %H:%M:%S')" "$(PRESET)"
	mkdir -p build/$(PRESET) && cmake --preset=$(PRESET)

build: config
	@printf "$(GREEN)%s Building with preset: %s$(RESET)\n" "$$(date '+%Y-%m-%d %H:%M:%S')" "$(PRESET)"
	cmake --build build/$(PRESET)
	ln -sf build/$(PRESET)/compile_commands.json .
```


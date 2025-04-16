# 安装 

## 安装基础依赖 

```shell
sudo apt update
sudo apt install build-essential git curl make gcc libncurses-dev
```

## 安装编译器

```
git clone https://github.com/richfelker/musl-cross-make.git

cat << EOF > config.mak
GCC_VER = 14.2.0
TARGET = mipsel-linux-musl
OUTPUT = /opt/musl-cross
GNU_SITE = https://mirrors.tuna.tsinghua.edu.cn/gnu/
EOF
```

> target 支持

- `aarch64[_be]-linux-musl`
- `arm[eb]-linux-musleabi[hf]`
- `i*86-linux-musl`
- `microblaze[el]-linux-musl`
- `mips-linux-musl`
- `mips[el]-linux-musl[sf]`
- `mips64[el]-linux-musl[n32][sf]`
- `powerpc-linux-musl[sf]`
- `powerpc64[le]-linux-musl`
- `riscv64-linux-musl`
- `s390x-linux-musl`
- `sh*[eb]-linux-musl[fdpic][sf]`
- `x86_64-linux-musl[x32]`


> Here Document

- 语法

```shell
命令 << 终止符
多行文本内容...
终止符

eg:

cat << EOF 
hello
world
EOF 

ssh user@host << EOF
ls -l
df -h
EOF

```

## 编译

```shell
make -j$(nproc)
sudo make install
echo 'export PATH="/opt/musl-cross/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 验证

```shell
/opt/musl-cross/bin/mipsel-linux-musl-c++ --version
```

# `with meson`

## 配置

```shell

cat << EOF > mips.ini
[constants]
toolchain = '/opt/musl-cross'
sysroot = toolchain + '/mipsel-linux-musl'

[binaries]
c = toolchain + '/bin/mipsel-linux-musl-cc'
cpp = toolchain + '/bin/mipsel-linux-musl-c++'
ar = toolchain + '/bin/mipsel-linux-musl-ar'
strip = toolchain + '/bin/mipsel-linux-musl-strip'

[properties]
sys_root = sysroot
debug = false
strip = true

[host_machine]
system = 'linux'
cpu_family = 'mips'
cpu = 'mipsel'
endian = 'little'

EOF
```

## 编译

```shell
meson setup target -Db_lto=true -Dbuildtype=release --cross-file mips.ini -Dstrip=true
meson compile -C target --verbose
meson install -C target --destdir=./install

```

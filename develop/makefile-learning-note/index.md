# Makefile/GCC编译选项 学习笔记


## 概述

Makefile 是 GNU make 使用的构建脚本，通过定义目标（target）、依赖（prerequisites）和命令（command）来描述如何从源文件生成目标文件。本文整理 Makefile 语法要点、GCC 常用选项及常见错误。

## Makefile 基本结构

```makefile
target: prerequisites
	command
```

| 部分 | 说明 |
|------|------|
| `target` | 目标，可以是文件或伪目标 |
| `prerequisites` | 执行 target 所需的文件或其他目标 |
| `command` | make 需要执行的 shell 命令，**必须以 Tab 开头** |

当 prerequisites 中有一个或多个文件比 target 新时，command 定义的命令会被重新执行。

## make 命令行选项

| 选项 | 说明 |
|------|------|
| `make -C dir` | 切换到目录 `dir` 下执行 make |
| `make -f file` | 指定 Makefile 文件（默认查找 `Makefile` / `makefile`） |

## GCC 常用编译选项

| 选项 | 说明 |
|------|------|
| `-o` | 指定输出文件名 |
| `-c` | 编译生成 `.o` 目标文件，不进行链接 |
| `-I` | 指定头文件（include）搜索路径 |
| `-O` | 优化选项 |
| `-Wall` | 开启常见警告 |
| `-Werror` | 将所有警告视为错误，遇到警告即停止编译 |
| `-L` | 指定链接库的搜索目录 |
| `-l` | 指定链接库名称，一般与 `-L` 成对使用 |
| `-fPIC` | 生成位置无关代码，共享库通常需要此选项 |
| `-E` | 仅做预处理 |
| `-g` | 生成调试信息 |
| `-shared` | 生成共享库 |
| `-std=gnu11` | 指定 C 语言标准（如 `c11`、`gnu99`） |
| `-pthread` | 编译与链接时启用 POSIX 线程支持 |
| `-static` | 静态链接（不依赖运行时 `.so`） |
| `-MMD -MP` | 生成依赖文件（`.d`），供 Makefile 自动追踪头文件变更 |

### 安全编译选项

现代 GCC 提供多种选项，可在编译/链接阶段降低缓冲区溢出、ROP 等攻击风险。以下选项分**编译阶段**（`CFLAGS`/`CXXFLAGS`）和**链接阶段**（`LDFLAGS`）。

#### 栈保护（Stack Protector）

| 选项 | 说明 |
|------|------|
| `-fstack-protector` | 为有栈帧的函数插入 canary，检测栈溢出 |
| `-fstack-protector-strong` | 保护范围更广（含数组、局部变量地址等），**推荐** |
| `-fstack-protector-all` | 保护所有函数，开销最大 |

#### 地址空间布局随机化（PIE）

| 选项 | 阶段 | 说明 |
|------|------|------|
| `-fPIE` | 编译 | 生成位置无关的目标代码 |
| `-pie` | 链接 | 生成 PIE 可执行文件，配合 ASLR 随机化加载地址 |

共享库本身已用 `-fPIC`；可执行文件需 `-fPIE` + `-pie` 才能启用 PIE。

#### 运行时强化（FORTIFY_SOURCE）

```makefile
CFLAGS += -O2 -D_FORTIFY_SOURCE=2
```

| 宏 | 说明 |
|----|------|
| `_FORTIFY_SOURCE=1` | 轻量检查，需 `-O1` 及以上 |
| `_FORTIFY_SOURCE=2` | 更严格（如 `memcpy` 长度校验），需 `-O1` 及以上，**推荐** |

对 glibc 中部分字符串/内存函数做编译期与运行时边界检查，需配合优化级别使用。

#### 链接器安全选项

通过 `-Wl,` 传递给 ld：

| 选项 | 说明 |
|------|------|
| `-Wl,-z,relro` | 启用 RELRO，将 GOT 等重定位段设为只读 |
| `-Wl,-z,now` | 立即绑定（BIND_NOW），与 `relro` 合用称 **Full RELRO**，防止 GOT 覆写 |
| `-Wl,-z,noexecstack` | 标记栈不可执行（NX） |
| `-Wl,-z,defs` | 共享库链接时，若有未定义符号则报错 |
| `-Wl,--as-needed` | 仅链接实际引用的库，减少多余依赖 |

#### 警告与代码质量

| 选项 | 说明 |
|------|------|
| `-Wall -Wextra` | 开启更多警告 |
| `-Wformat -Wformat-security` | 检查 `printf` 类格式字符串，防止格式串漏洞 |
| `-Werror=format-security` | 将格式安全警告视为错误 |
| `-fno-omit-frame-pointer` | 保留帧指针，便于 backtrace（略增开销） |
| `-fvisibility=hidden` | 默认隐藏符号，仅导出需对外接口（共享库常用） |

#### 推荐组合示例

**可执行文件（Release + 安全加固）：**

```makefile
CFLAGS  += -O2 -pipe -Wall -Wextra \
           -fstack-protector-strong \
           -D_FORTIFY_SOURCE=2 \
           -fPIE

LDFLAGS += -pie \
           -Wl,-z,relro -Wl,-z,now \
           -Wl,-z,noexecstack
```

**共享库：**

```makefile
CFLAGS  += -O2 -fPIC -fstack-protector-strong -D_FORTIFY_SOURCE=2 -fvisibility=hidden
LDFLAGS += -shared -Wl,-z,defs
```

**调试构建**时通常关闭 `-D_FORTIFY_SOURCE` 或降低优化，并保留 `-g`；发布前再启用完整安全选项。

查看二进制是否启用 PIE / RELRO 等：

```bash
checksec --file=./main          # 需安装 checksec
readelf -h ./main | grep Type   # Type: DYN 表示 PIE
readelf -l ./main | grep GNU_RELRO
```

### 编译示例

只编译不链接（生成 `.o`）：

```bash
gcc -c -fPIC add.c
```

生成共享库：

```bash
gcc -shared -fPIC -o libadd.so add.o
```

## 动态库链接与 rpath

链接可执行文件或共享库时，除了 `-L`（指定**链接阶段**搜索目录）和 `-l`（指定库名）外，还需要考虑**运行时**如何找到 `.so` 文件。

### 库搜索路径概览

| 方式 | 作用阶段 | 说明 |
|------|----------|------|
| `-L dir` | 链接时 | 告诉链接器去哪里找 `libxxx.so` 或 `libxxx.a` |
| `-lxxx` | 链接时 | 链接名为 `libxxx.so` / `libxxx.a` 的库 |
| `-Wl,-rpath,dir` | 运行时 | 将路径写入可执行文件/库的 ELF，运行时动态链接器按此搜索 |
| `-Wl,-rpath-link,dir` | 链接时 | 仅链接阶段有效，解析**间接依赖**的 `.so`，不写入 ELF |
| `LD_LIBRARY_PATH` | 运行时 | 环境变量，临时指定额外搜索路径 |
| `/etc/ld.so.conf` + `ldconfig` | 运行时 | 系统级配置，更新 `/etc/ld.so.cache` |

运行时搜索顺序（简化）大致为：`RPATH/RUNPATH` → `LD_LIBRARY_PATH` → `/etc/ld.so.cache` → 默认目录（如 `/lib`、`/usr/lib`）。

### rpath 与 runpath

通过 `-Wl,-rpath,DIR` 可将目录嵌入 ELF 的 `DT_RPATH` 或 `DT_RUNPATH` 字段：

```bash
gcc -o main main.o -L./lib -ladd -Wl,-rpath,/opt/myapp/lib
```

| 字段 | 常见写法 | 说明 |
|------|----------|------|
| `DT_RPATH` | `-Wl,-rpath,DIR`（旧行为） | 优先级较高，可能覆盖 `LD_LIBRARY_PATH` |
| `DT_RUNPATH` | 同上，配合 `--enable-new-dtags` | 较新的默认行为，优先级低于 `LD_LIBRARY_PATH` |

```bash
# 写入 DT_RUNPATH（推荐，便于开发时用 LD_LIBRARY_PATH 覆盖）
gcc -o main main.o -L./lib -ladd \
    -Wl,--enable-new-dtags -Wl,-rpath,/opt/myapp/lib

# 强制写入 DT_RPATH（旧行为）
gcc -o main main.o -L./lib -ladd \
    -Wl,--disable-new-dtags -Wl,-rpath,/opt/myapp/lib
```

### 使用 `$ORIGIN` 指定相对路径

部署时库与可执行文件在同一目录树内时，可用 `$ORIGIN` 表示「可执行文件所在目录」：

```bash
gcc -o bin/main main.o -L./lib -ladd \
    -Wl,-rpath,'$ORIGIN/../lib'
```

Makefile 中注意对 `$` 转义：

```makefile
LDFLAGS += -Wl,-rpath,'$$ORIGIN/../lib'
```

### rpath-link（链接间接依赖）

链接 `main` 时若依赖 `libA.so`，而 `libA.so` 又依赖同目录下的 `libB.so`，链接器在链接阶段可能找不到 `libB.so`。此时用 `-rpath-link` 仅帮助**完成链接**，路径不会写入最终 ELF：

```bash
gcc -o main main.o -L./lib -lA \
    -Wl,-rpath-link,./lib
```

### 查看已嵌入的路径

```bash
readelf -d main | grep -E 'RPATH|RUNPATH'
ldd main
```

### Makefile 示例

```makefile
LIB_DIR := $(CURDIR)/lib
INSTALL_LIB := /opt/myapp/lib

CFLAGS  += -I$(LIB_DIR)/../include
LDFLAGS += -L$(LIB_DIR)
LDLIBS  += -ladd

# 开发阶段：相对路径，便于本地运行
LDFLAGS += -Wl,--enable-new-dtags -Wl,-rpath,'$$ORIGIN/../lib'

# 或安装阶段：绝对路径
# LDFLAGS += -Wl,-rpath,$(INSTALL_LIB)

main: main.o
	$(CC) -o $@ $^ $(LDFLAGS) $(LDLIBS)
```

### 常见问题

**`error while loading shared libraries: libxxx.so: cannot open shared object file`**

- 链接时缺少 `-L` 或 `-l`
- 运行时找不到 `.so`：设置 `LD_LIBRARY_PATH`、配置 rpath，或将库安装到系统路径并执行 `ldconfig`

**开发环境与部署环境路径不一致**

- 开发机用 `LD_LIBRARY_PATH=./lib ./main` 快速验证
- 发布时用 `-Wl,-rpath,'$ORIGIN/../lib'` 或安装到固定目录并写入 rpath

**与「DSO missing from command line」的区别**

- rpath 解决的是**运行时找不到库文件**
- DSO missing 是**链接时**未声明对某个 `.so` 的依赖，需补 `-l` 选项

## Makefile 内置变量

| 变量 | 说明 |
|------|------|
| `$@` | 当前规则的目标（target） |
| `$<` | 依赖中的第一个文件 |
| `$^` | 依赖中的所有文件 |

## 赋值与宏定义

| 写法 | 说明 |
|------|------|
| `:=` | 立即赋值，覆盖之前的值 |
| `?=` | 若变量未被赋值，则赋予默认值 |
| `+=` | 追加赋值 |

在 CFLAGS 中定义宏：

```makefile
CFLAGS += -D_CBB
```

## Makefile 函数

### `$(subst FROM, TO, TEXT)`

将字符串 `TEXT` 中的子串 `FROM` 替换为 `TO`。

### `$(word N, TEXT)`

取字符串 `TEXT` 中第 `N` 个单词（从 1 开始）。

- `N = 0` 会报错
- `N` 大于单词总数时返回空字符串

## 多文件链接

```makefile
main-objs := a.o b.o c.o
```

将 `a.c`、`b.c`、`c.c` 编译后链接生成 `main`。

## 内核模块相关变量

| 变量 | 说明 |
|------|------|
| `obj-m` | 编译为可加载的内核模块 |
| `obj-y` | 编译进内核 |
| `obj-n` | 不编译 |

## include 指令

`include` 与 C/C++ 的 `#include` 类似，make 启动时会读取 include 指定的其他 Makefile 并展开到当前文件。

搜索顺序：

1. 当前目录
2. 命令行 `-I` / `--include-dir` 指定的目录
3. `<prefix>/include`（如 `/usr/local/include`、`/usr/include`）

若找不到文件，make 默认会报 warning；全部读取完成后会重试，仍失败则 fatal error。

使用 `-include`（或 `sinclude`）可在文件缺失时不报错、继续执行：

```makefile
-include other.mk
```

仅当无法完成最终目标重建（必需目标找不到规则）时才会 fatal error 并退出。

## 调试与分析工具

### Sanitizer 与代码追踪

GCC / Clang 提供 **Sanitizer**（`-fsanitize=*`），在编译期插入检测代码，**运行时**自动报告内存错误、未定义行为、数据竞争等问题，并打印带源码行号的调用栈。适合开发阶段替代「反复 GDB + core」的排查方式。

> Sanitizer 会显著增加内存/CPU 开销，**仅用于 Debug 构建**，不要带入 Release 发布包。

#### 通用用法

编译与链接需**同时使用**相同的 `-fsanitize` 选项，并建议加 `-g`、`-fno-omit-frame-pointer` 以获得清晰 backtrace：

```makefile
# Makefile 中常见写法：单独 debug / sanitize 目标
SAN_FLAGS := -fsanitize=address -fno-omit-frame-pointer -g -O1

debug: CFLAGS += $(SAN_FLAGS)
debug: LDFLAGS += $(SAN_FLAGS)
debug: main
```

| 选项 | 说明 |
|------|------|
| `-g` | 保留调试符号，报告可显示文件名与行号 |
| `-O0` / `-O1` / `-Og` | Sanitizer 推荐 `-O1` 或 `-Og`；过高优化可能误报或影响检测 |
| `-fno-omit-frame-pointer` | 保留帧指针，backtrace 更完整 |
| `-fsanitize-recover=...` | 部分 UBSan 类错误检测到后继续运行（默认 UBSan 直接 abort） |

#### AddressSanitizer（ASan）

检测**堆/栈/全局缓冲区溢出**、use-after-free、double-free 等内存错误。

```bash
gcc -g -O1 -fsanitize=address -fno-omit-frame-pointer -o main main.c
./main
```

典型输出会指出错误类型、读写地址、分配/释放栈（allocation stack）及发生位置。

常用环境变量：

```bash
export ASAN_OPTIONS=halt_on_error=1:abort_on_error=1:detect_leaks=1:symbolize=1
```

| 变量 | 说明 |
|------|------|
| `halt_on_error=1` | 遇到第一个错误即停止 |
| `detect_leaks=1` | 启用 LeakSanitizer（默认随 ASan 开启） |
| `symbolize=1` | 将地址解析为函数名（需 `llvm-symbolizer` 或在 PATH 中） |
| `log_path=/tmp/asan` | 将报告写入文件 |

#### UndefinedBehaviorSanitizer（UBSan）

检测 C/C++ **未定义行为**：整数溢出、空指针解引用、越界枚举、对齐错误等。

```bash
gcc -g -O1 -fsanitize=undefined -o main main.c
```

可只启用部分检查：

```bash
-fsanitize=signed-integer-overflow,null,alignment,bounds
```

```bash
export UBSAN_OPTIONS=print_stacktrace=1:halt_on_error=1
```

#### ThreadSanitizer（TSan）

检测**数据竞争**（data race）与部分死锁相关场景，适用于多线程程序。

```bash
gcc -g -O1 -fsanitize=thread -o main main.c -pthread
```

> TSan 与 ASan **不能同时**启用。TSan 内存开销约为原程序的 5～10 倍。

```bash
export TSAN_OPTIONS=halt_on_error=1:history_size=7
```

#### LeakSanitizer（LSan）

检测**内存泄漏**，通常随 ASan 自动启用，也可单独使用：

```bash
gcc -g -O1 -fsanitize=leak -o main main.c
```

程序退出时汇总未释放的分配及分配调用栈。

#### 其他 Sanitizer（了解）

| 选项 | 名称 | 说明 |
|------|------|------|
| `-fsanitize=memory` | MSan | 检测未初始化内存读取；GCC 支持有限，Clang 更常用 |
| `-fsanitize=hwaddress` | HWASan | 硬件辅助 ASan，多见于 ARM64 |
| `-fsanitize=pointer-compare` / `pointer-subtract` | — | 非法指针对比/减法 |

#### Sanitizer 对照

| Sanitizer | 主要检测 | 典型场景 | 可否与 ASan 同开 |
|-----------|----------|----------|------------------|
| ASan | 内存越界、UAF | 通用 C/C++ 调试 | — |
| UBSan | 未定义行为 | 算术/类型/对齐问题 | 可以 |
| TSan | 数据竞争 | 多线程 | 否 |
| LSan | 内存泄漏 | 退出时泄漏汇总 | 随 ASan 默认开启 |

#### Makefile 示例：Debug / Sanitize 目标

```makefile
CC      := gcc
CFLAGS  := -Wall -Wextra -std=gnu11
LDFLAGS :=

main: main.o
	$(CC) $(LDFLAGS) -o $@ $^

# 常规调试（GDB）
debug: CFLAGS  += -g -Og
debug: LDFLAGS += -g
debug: main

# ASan 构建
asan: CFLAGS  += -g -O1 -fsanitize=address -fno-omit-frame-pointer
asan: LDFLAGS += -fsanitize=address
asan: main

# UBSan 构建
ubsan: CFLAGS  += -g -O1 -fsanitize=undefined -fno-omit-frame-pointer
ubsan: LDFLAGS += -fsanitize=undefined
ubsan: main

# TSan 构建（多线程）
tsan: CFLAGS  += -g -O1 -fsanitize=thread -fno-omit-frame-pointer
tsan: LDFLAGS += -fsanitize=thread -pthread
tsan: main
```

运行示例：

```bash
make asan
ASAN_OPTIONS=symbolize=1:halt_on_error=1 ./main
```

#### 其他编译期追踪手段

| 选项 / 工具 | 说明 |
|-------------|------|
| `-pg` | 插入 gprof 性能分析探针，运行后 `gprof` 查看热点 |
| `-fprofile-arcs -ftest-coverage` + `gcov` | 代码覆盖率统计 |
| `-finstrument-functions` | 在每个函数入口/出口插入钩子，自定义追踪逻辑 |
| `-fsanitize=address` + GDB | ASan 报错后可在 `abort` 前附加 GDB：`ASAN_OPTIONS=abort_on_error=0 ./main` |

### GDB

#### 编译选项

| 命令/选项 | 说明 |
|-----------|------|
| `-g` | 编译时加入调试信息（使用 GDB 分析 core 或源码调试时需要） |
| `strip` | 去除可执行文件中的调试信息，减小体积，但会无法调试 |

#### 使用 core dump 定位崩溃

进程异常退出时，内核可将内存镜像写入 core 文件，再用 GDB 结合可执行文件分析调用栈。完整流程如下。

**1. 开启内核编译选项**

通过 `menuconfig` 开启 ELF core dump 支持：

```
General setup
  └── Configure standard kernel features (for small systems)
        └── Enable ELF core dumps
```

也可直接在内核配置文件中启用对应选项，然后**重新编译内核**，并**重新编译待调试的应用**（需带 `-g`）。

**2. 启动后配置运行环境**

```bash
ulimit -c unlimited
echo "/tmp/core-%e-%p" > /proc/sys/kernel/core_pattern
```

| 配置 | 说明 |
|------|------|
| `ulimit -c unlimited` | 取消 core 文件大小限制 |
| `core_pattern` | 指定 core 文件路径与命名格式；`%e` 为可执行文件名，`%p` 为进程 PID |

**3. 启动待调试进程**

以 `mapd` 为例：先结束已有进程，将二进制复制到可写目录后手动启动：

```bash
killall mapd
cp /userfs/bin/mapd /tmp/
/tmp/mapd -G /etc/wts_bss_info_config \
          -I /etc/mapd_cfg.txt \
          -O /etc/mapd_strng.conf \
          -c client_db.txt > /dev/console &
```

**4. 复现问题并确认 core 文件**

复现崩溃后，用 `ps` 确认进程已退出。若配置正确，`/tmp/` 下应出现类似 `core-mapd-12345` 的 core 文件。

**5. 启动 GDB 分析**

将 GDB 传到设备（嵌入式环境常用 TFTP）：

```bash
cd /tmp/
tftp -gr gdb 192.168.1.xx
chmod +x gdb
./gdb mapd /tmp/core-mapd-12345
```

**6. 查看 backtrace**

```gdb
(gdb) bt
```

`bt`（backtrace）会打印崩溃时的函数调用栈，据此定位出错代码路径。常用补充命令：

| 命令 | 说明 |
|------|------|
| `bt full` | 显示调用栈及局部变量 |
| `frame N` | 切换到第 N 帧 |
| `list` | 查看当前帧附近源码 |
| `info registers` | 查看寄存器状态 |

> 分析 core 时，GDB 使用的可执行文件应与生成 core 时的版本一致（同一编译产物），否则符号和行号可能对不上。

### objdump

反汇编与分析目标文件：

```bash
objdump -t file    # 显示符号表
```

### readelf

查看 ELF 文件结构：

```bash
readelf -d file    # 显示动态段信息
```

## 常见错误

### missing separator

Makefile 中的命令必须以 **Tab** 开头，不能用空格。

### skipping incompatible

链接库时，库文件与当前平台/架构不匹配（例如 32 位库链接 64 位程序）。

### DSO missing from command line

动态库依赖未显式链接。例如：

```
libA.so ---ld---> libB.so
main    ---ld---> libA.so
```

若 `main` 调用了 `libB.so` 中的函数，但链接时只写了 `-lA`、没有 `-lB`，就会报此错。生成 `main` 时需要显式加上 `-lB`（或 `-lB -lA` 等完整依赖）。


---

> 作者: [Charles Y](https://github.com/devCharl/devcharl.github.io)  
> URL: https://devcharl.github.io/develop/makefile-learning-note/  


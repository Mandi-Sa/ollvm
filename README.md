# 阿菌•未霜 Obfuscation LLVM Toolchain

![Release](https://img.shields.io/github/v/release/Mandi-Sa/ollvm)

一套带有多种混淆 Pass 的完整 LLVM/Clang 工具链预构建发行版，开箱即用，无需手动编译。

---

## 目录

- [简介](#简介)  
- [特性](#特性)  
- [混淆功能列表](#混淆功能列表)  
- [快速开始](#快速开始)  
  - [下载预构建包](#下载预构建包)  
  - [环境配置](#环境配置)  
- [使用示例](#使用示例)  
  - [开启全部混淆](#开启全部混淆)  
  - [单函数注解控制 (Annotate)](#单函数注解控制-annotate)  
- [注意事项](#注意事项)  
- [Credits](#credits)  

---

## 简介

本项目提供了一个基于 LLVM/Clang 的预构建工具链，内置多种代码混淆（Obfuscation）Pass，适合对 C/C++ 二进制进行增强防护。你只需下载合适平台的压缩包，配置好环境变量，即可使用。

---

## 特性

- 完整 LLVM/Clang 环境（包含 `clang`、`clang++`、`llc`、`llvm-objdump` 等工具）  
- 多种控制流和数据流混淆手段：伪控制流、扁平化、指令替换、字符串混淆、基本块拆分、间接分支/调用/全局变量混淆  
- 支持按函数级别注解精细启停  
- 支持命令行一键开启任意组合混淆  
- 发布版即用，无需本地编译 LLVM  
- 尽可能的提供最大化优化（Linux版本带有Full LTO、PGO、BOLT）

---

## 混淆功能列表

| 开关            | 描述                                               | 默认值      |
| --------------- | -------------------------------------------------- | ----------- |
| `-mllvm -bcf`   | 伪控制流 (Bogus Control Flow)                      |              |
| `-mllvm -bcf_prob=<1–100>` | 伪控制流混淆概率（%）                    | 70          |
| `-mllvm -bcf_loop=<n>`     | 伪控制流重复次数                         | 2           |
| `-mllvm -fla`   | 控制流扁平化 (Flattening)                          |              |
| `-mllvm -sub`   | 算术/逻辑指令替换 (add/and/sub/or/xor)              |              |
| `-mllvm -sub_loop=<n>`     | 指令替换次数                            | 1           |
| `-mllvm -sobf`  | 字符串混淆 (仅窄字符，仅命令行支持，不支持annotation) |              |
| `-mllvm -split` | 基本块拆分                                         |              |
| `-mllvm -split_num=<n>`    | 每个基本块拆分数量                       | 3           |
| `-mllvm -ibr`   | 间接分支 (Indirect Branch)                         |              |
| `-mllvm -icall` | 间接调用 (Call Register)                           |              |
| `-mllvm -igv`   | 间接全局变量 (Indirect Global Variable)             |              |
| `-mllvm -fncmd` | 函数名混淆(不推荐，建议使用annotation)               |              |

---

## 快速开始

### 下载预构建包

1. 访问 Releases 页面：  
   https://github.com/Mandi-Sa/ollvm/releases  
2. 根据操作系统选择对应压缩包（Windows/Linux/macOS）。  
3. 解压到任意目录，例如：  
   ```bash
   7zz x ollvm-21.1.0-linux-amd64_lto+pgo+bolt.7z -o ~/ollvm
   ```

### 环境配置

将解压目录加入 `PATH`，以便直接调用 `clang`、`clang++` 等工具：
   ```bash
   export PATH="$HOME/ollvm/bin:$PATH"
   ```

### 使用示例

假设你已有一个 `main.c`，下面演示如何开启混淆并编译：
   ```bash
   clang -O2 \
     -mllvm -fla \
     -mllvm -bcf -mllvm -bcf_prob=80 -mllvm -bcf_loop=3 \
     -mllvm -sub -mllvm -sub_loop=2 \
     -mllvm -split -mllvm -split_num=5 \
     -mllvm -sobf \
     -mllvm -ibr -mllvm -icall -mllvm -igv \
     main.c -o main_obf
   ```

#### 开启全部混淆

   ```bash
   clang -O2 \
     -mllvm -fla \
     -mllvm -bcf \
     -mllvm -bcf_prob=80 \
     -mllvm -bcf_loop=3 \
     -mllvm -sobf \
     -mllvm -icall \
     -mllvm -ibr \
     -mllvm -igv \
     -mllvm -sub \
     -mllvm -sub_loop=3 \
     -mllvm -split \
     -mllvm -split_num=5 \
     main.c -o main_all_obf
   ```

#### 单函数注解控制 (Annotate)

在不想全局打开某些 Pass 时，可在源码中按函数级别开启/关闭。示例：

   ```cpp
   // 强制开启 fla、bcf、igv、ibr、icall、sub、split
   int __attribute__((annotate("fla bcf igv ibr icall sub split")))
   test_enable_all() {
       printf("Hello OLLVM\n");
       return 0;
   }

   // 强制关闭 fla、bcf、split
   int __attribute__((annotate("nofla nobcf nosplit")))
   test_disable_some() {
       printf("No FLA/BCF/SPLIT here\n");
       return 0;
   }
   ```

## 注意事项

- 此分支的工具链使用基于glibc 2.39的Ubuntu 24.04 LTS构建，无法保证比该版本更旧的发行版的兼容性。不支持其他 libc 实现（例如 musl）。
- `-fla` 和 `-bcf` 会显著增加编译时间，并可能导致大型函数无法生成可执行代码。

## Credits

[LLVM](https://github.com/llvm/llvm-project "LLVM")

[SsageParuders](https://github.com/SsageParuders/SsagePass "SsageParuders")

[wwh1004](https://github.com/wwh1004/ollvm-16 "wwh1004")

[DreamSoule](https://github.com/DreamSoule/ollvm17 "DreamSoule")

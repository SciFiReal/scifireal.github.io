---

title: "基础篇01-VS/Qt开发环境配置以及编译运行原理"

date: 2026-10-04T23:57:00+08:00

draft: false

categories: ["C++"]

tags: ["C++","Qt"]

summary: "基础篇01-VS/Qt开发环境配置以及编译运行原理"

toc: true

---

## 1. Visual Studio

VS是专门用来写C/C++项目的大型编辑器。在学习C++语法时，很多人不是被语法劝退，而是被杂乱环境、编辑器不兼容、编译报错、运行报错等原因折磨。因为VS的工具连比较多，大家可能不太能理解这整个流程。本篇深入拆解C++从编译-链接-运行的完整底层流程，还有配置工具链。

VS的具体安装教程其询问大模型，需要注意以下两点。

1. 必须安装的组件：

- 安装组件仅需要勾选**使用C++的桌面开发**组件，该组件包含C++编译器，调试器，标准库等等，完全满足C++的所有学习。

2. 规范项目创建：

- 新建空项目，不要去新建自带模板冗余代码
- 项目、路径、文件名全部使用英文、小写、无特殊符号，避免没有必要的麻烦
- 右键项目属性，确保编译器为ISO C++11及以上标准

## 2. VS中编译流程

我们直接在VS中新建C++桌面项目，直接得到 `xxx.sln + xxx.vcxproj`。`.sln`是VS的解决方案文件，本质不是代码，不是编译产物，只是一个项目清单，容器描述文件，类似一本书的目录，记录多个项目，比如主程序项目、静态库项目等等。`.vcxproj`类似每一章的完整内容（编译任务清单）。最后交给`MSBuild`编译构建工具来调用`MSVC编译工具链`来编译项目。这是VS默认的编译流程： 

> VS新建项目 &rarr; 生成`.sln和.vcxproj` &rarr; `MSBuild`编译构建工具 &rarr; `MSVC`编译工具链 &rarr; `.exe`

---

## 3. C/C++ 项目编译构建体系

### 3.1 CMake

目前太多数项目都会采用CMake来构建项目，因此以CMake来进行分类。

`CMake`是什么？是跨平台的构建描述生成器。你写一份`CMakeLists.txt`，`CMake`根据你的环境，自动生成对应平台的构建脚本，本身不编译代码。好处就是能够跨平台，一份配置，到处构建，方便换编译器。

这个构建脚本是什么呢？主要有三种`sln+vcxproj`、`Makefile`、`ninja`。

- `-G Ninja` → 生成`build.ninja`，交给`Ninja`
- `-G "MinGW Makefiles"`→ 生成`Makefile`，交给`mingw32-make`
- `-G "Visua1 Studio 17 2022"`→ 生成`.s1n` （总的清单）和`.vcxproj`（单个文件清单），交给`MSBuild`

### 3.2 构建工具

`Ninja`，`mingw32-make`，`MSBuild`是**构建工具**，作用是把源代码编译、链接，最终生成可执行程序`.exe/库`，只是出身，使用编译器，平台不一样。构建工具不是编译器，编译器是把.cpp翻译成机器码，构建工具是处理依赖，调度编译任务。

- `Ninja`是现代轻量、高速建构工具，是`CMake`最常用的后端，适合大型C/C++项目，只是调用器，会调用gcc/clang/cl.exe（MSVC）都可以。
- `mingw32-make`就是Windows版的GNU Make，在不同平台上叫法不一样，在Linux上直接`make`，Windows MinGW下叫`mingw32-make`；读取`Makefile`，调用`gcc/g++`(MinGW编译器)编译C/C++代码；是属于老工具，大项目下并行编译弱、依赖扫描慢。
- `MSBuild`是微软官方的构建工具，配合MSVC（cl.exe）。

### 3.3 工具链

`MinGW-w64(MinGW64)`和`MSVC`是两套完整C/C++工具链；`g++`、`cl.exe`是各自工具链的C++编译器前端。工具链包含一整套配套程序，有编译器，预处理器、汇编器、链接器、标准库、头文件、系统库、运行时。

1. `MinGW-w64(MinGW64)`下有：

- `gcc/g++`：C/C++对应的编译器
- `as.exe`：汇编器
- `ld.exe`：链接器
- 头文件 + Windows API 导入库 + **mingw-w64 版标准库 (libstdc++)**
- 还有 `mingw32-make`（GNU Make，**不属于编译器本身，只是配套构建工具**）

特点是：可以搭配 `mingw32-make` 或者` Ninja` 作为构建后端



2. `MSVC`下有：

- `cl.exe`：MSVC 的 C/C++ 编译器（C 和 C++ 共用这一个 exe）

- `ml/ml64.exe`：微软汇编器

- `link.exe`：微软链接器

- 微软头文件、CRT（微软 C 运行时库）、STL（MSVC STL）、Windows SDK

- 配套构建工具：`MSBuild`

  

> Clang：编译器，是独立于`MinGW`和`MSVC`工具链的前端，和`g++`、`cl.exe`作用一样

> ==特别是构建Qt项目的环境配置，会用到工具链、构建方式、编译器都是可以组合搭建的完整的项目编译流程==，例如CMake + MSVC + Ninja + Qt (MSVC 版)，CMake + MinGW + Ninja + Qt (MinGW 版)。详细过程如下：
>
> .cpp源码 +　CMakeLists.txt
> 	↓ Cmake -G Ninja
> build.ninja(任务清单)
> 	↓ Ninja(读取清单)
> 调用 MinGW-w64(工具链) 的 g++.exe + ld.exe等
> 	↓
> exe

---

## 4. C/C++ 完整编译流程是什么呢?

**通常把预处理、编译、汇编统称为编译**，注意调用g++/clang++/cl.exe会直接一次性走完预处理、编译、汇编、链接。

```text
.cpp
↓
预处理：.cpp预处理后的源码文件.i：处理宏、`#include`、注释删除、得到纯源码文本
↓
编译: 编译器（g++ /cl.exe/clang++）得到.s：做语法分析、词法分析，C++代码转汇编指令
↓
汇编：汇编器把汇编指令转为机器码，输出.obj(MSVC)/.o(MinGW/GCC/Clang)目标文件
↓
链接：连接器（ld.exe/link.exe）输出.exe：合并多个目标文件+库.lib/.a/.dll，生成可执行程序
↓
.exe：Windows下可直接运行的程序
```

> `.a`是GCC静态库；`.lib`是MSVC静态库；`.dll`是Windows动态库，运行时加载
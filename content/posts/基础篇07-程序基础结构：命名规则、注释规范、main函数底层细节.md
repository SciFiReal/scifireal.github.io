---
title: "基础篇07-程序基础结构：命名规则、注释规范、main函数底层细节"

date: 2026-10-05T00:10:28+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "基础篇07-程序基础结构：命名规则、注释规范、main函数底层细节"

toc: true
---



## 1. 头文件`#include`底层原理与使用规范

`#include`的本质是：预处理阶段，将指定的文件的内容复制到当前文件。示例如下：

function.h 声明：

```c++
#pragma once   // 作用是：防止这个头文件被多次include，避免重复定义。本质是编译器识别这个头文件，之后再遇到#include就直接跳过。例如如果你在头文件中实现了函数，被多次#include之后就会出现重复定义的错误

int add(int a, int b);
```

function_finhs.cpp 实现：

```c++
#include "function.h"

int add(int a, int b)
{
	return a + b;
}
```

main.cpp 调用：

```c++
#include <iostream>
#include "function.h"

int main() {
	std::cout << add(3, 4) << std::endl;  // 7
	return 0;
}
```

> 你发现头文件有两种引入方式：一种是<...>，另一种是"..."，比如
>
> #include <iostream>  与 #include "function.h"
>
> <...>：表示系统标准库头文件，为何不用.h后缀是为了与C语言导入标准库进行区分；优先从编辑器库目录中查找；
>
> "..."：优先从当前项目目录下查找，再去系统目录查找；一般是自己编写的头文件；

---

## 2. main函数深入讲解

C++规定一个合法的可执行程序，有且只有一个int main() 入口函数，不存在多个自定义入口。

有两种标准的写法：

```c++
// 写法1：无参标准写法（工业小程序、工具程序首选）
int main() 
{
	return 0;
}

// 写法2：带参标准写法（工控上位机、可执行程序、带启动参数项目）
int main(int argc, char* argv[])
{
	return 0;
}
```

如何来理解写法2呢？

`argc` = argument count 参数个数

`argv` = argument vector 参数数组（C风格字符串数组）

在cmd控制台输入

```bash
test.exe 100 hello
```

相当于：argc = 3

argv[0] = "text.exe"

argv[1] = "100"

argv[2] = "hello"

第二种写法适合需要外部传参，argv[0]永远是程序的名字exe文件名，参数从argv[1]开始。

---

## 3. 命名规范

合法命名：

1. 变量名由*字母*、*数字*、*下划线*组成；

2. 数字不能作为变量的首字母；

3. Device和device是两个不同的变量；

4. 不能用C++中关键字、保留字作为自定义变量名

规范命名：

1. 变量名：小驼峰（userScore）或下划线（user_score）命名，语义化、见名知意；还有一种常量或宏定义字母需要全部大写；
2. 函数名：动作语义，以动词开头，小驼峰 getScore ()，大驼峰 GetScore ()，下划线 get_score ()；
3. 禁止拼音、单字母无意义命名，禁止中英文混搭
4. 不同领域的相关变量有统一语义化，例如工业设备：sn，batch，errorCode；

---

## 4. 注释规范

```c++
// 这是单行注释

/*
多行注释
适合函数、类、功能说明
*/
```


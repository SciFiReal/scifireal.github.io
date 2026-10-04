---
title: "基础篇05-循环结构（for-while-do while）"

date: 2026-10-05T00:08:41+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "基础篇05-循环结构（for-while-do while）"

toc: true
---



## 1. while循环

while循环：先判断条件是否为真，若为真，则循环

```c++
// 变量表达式
while(条件表达式)
{
	// 循环体：重复代码执行
	// 必须更新循环变量，否则会死循环
}
```

```c++
int a = 1;
while(a < 5)
{
  std::cout << "第" << a << "次"<< std::endl;
  a++;
}
```

输出结果：

```bash
第1次
第2次
第3次
第4次
```

---

## 2. do-while循环

do-while：必须先执行一次，再判断循环条件

```c++
// 变量表达式
do
{
	// 循环体
  // 必须更新循环变量，否则会死循环
}(循环条件表达式)
```

```c++
int a = 1;
do
{
	std::cout << "第" << a << "次" << std::endl;
	a++;
} while (a < 5);
```

---

## 3. for循环

for循环将变量初始化、循环条件、更新变量全部整合再一起，可读性最强，也是**数组遍历唯一推荐写法**。

```c++
for(初始化表达式；循环条件；变量更新)
{
	// 循环体
}
```

```c++
int a = 1;

for (int a = 1; a < 6; a++)
{
	std::cout << "第" << a << "次" << std::endl;
}
```

输出结果：

```
第1次
第2次
第3次
第4次
第5次
```

---

## 4. break与continue：循环两大跳转关键字（关键点）

break的作用是跳出当前循环，终止当前所在循环

continue的作用是跳过本次循环，直接进入到下一次

while/do-while循环中使用continue，必须把变量更新写在continue前方，否则会进入死循环

用for循环来展示两者的区别

```c++
int a = 1;

for (int a = 1; a < 6; a++)
{
	std::cout << "第" << a << "次" << std::endl;
	if (a == 3)
	{
		break;   // break跳出当前所在for循环
	}
}

---结果
第1次
第2次
第3次
```

```c++
int a = 1;

for (int a = 1; a < 6; a++)
{
	if (a == 3)
	{
		continue;   // 调用a=3这次循环
	}
	std::cout << "第" << a << "次" << std::endl;
}

---结果
第1次
第2次
第4次
第5次
```


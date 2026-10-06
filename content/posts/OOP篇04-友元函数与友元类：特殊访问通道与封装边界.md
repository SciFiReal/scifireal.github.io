---
title: "OOP篇04-友元函数与友元类：特殊访问通道与封装边界"

date: 2026-10-06T17:50:07+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇04-友元函数与友元类：特殊访问通道与封装边界"

toc: true
---



> 为什么需要友元函数和友元类呢？

`C++`类中`private`只有本类内部能访问，这是类的封装，保护数据。但如果遇到有一个外部函数，逻辑上与这个类高度绑定，频繁要操作它的私有成员。如果不用友元，只能写一堆`get/set`接口。为了维护代码的简洁性，用友元来给特定外部代码放行，不用每次都走公开接口。

> 一句话总结：类的封装默认是对外隐藏私有成员；友元是给定函数/类，让它直接读/写这个类的`private/protected`成员，但友元不属于这个类。

> 友元有三种形式：友元函数，友元成员函数，友元类

---

## 1. 友元函数

```c++
class Box{
private:
    int width;
public:
    Box(int w):width(w){}
    // 声明友元函数，不是成员函数！
    friend void printWidth(Box b);
};
	
// 友元函数，不属于Box类，没有this指针
void printWidth(Box b){
    cout << b.width; // 可以直接访问 private！
}
```

> 可以看出：声明友元函数`friend`是在类内部，实现在类外面；
>
> 注意几点：一是友元函数不是成员函数没有`this`；二是友元不继承、不传递；三是权限单向：`printWidth` 能访问Box的私有；Box不能自动访问 `printWidth` 的东西。

> 什么场景下使用友元函数呢？

运算符重载，下一章会讲到。

---

## 2. 友元类

```c++
class A{
private:
    int secret = 100;
    friend class B; // B是A的友元类
};

class B{
public:
    void show(A a){
        cout << a.secret; // B可以直接访问A私有成员
    }
};
```

> A类定义`firend class B`，B 类的所有成员函数，都可以访问 A 类的 private/protected。

> 需要注意以下几点：

- A类的友元是B，A类不能访问B类的私有成员；A把B当朋友，B不把A当朋友
- 不传递，A类的友元是B，B的友元是C，C不是A的友元；朋友的朋友不是朋友
- 不继承，A类的友元是B，A类的子类不会自动拥有友元B类

> 什么场景下使用友元类？

B类中有很多方法需要使用到A类的私有/保护成员

---

## 3. 友元成员函数

有类A，类B；让类B中的某一个成员函数，可以访问A的private成员；类B中其它成员函数没有这个权限。

```c++
#include <iostream>
using namespace std;

// 1. 【前向声明】告诉编译器有个类B
class B;

// 2. 定义B类
class B {
public:
    void showA(); // 只是声明，不实现
  	void test();
};

class A {
private:
    int data = 999;
    // ✅ 3. 友元成员函数：授权 B 的 showA() 访问A私有成员
    friend void B::showA();
};

// 4. 现在A已经定义完毕，可以实现B::showA
void B::showA() {
    A a;
    cout << a.data << endl; // 有权访问A的private
}

void B::test() {
    A a;
    // cout << a.data; // ❌ 编译报错！test没有被声明为A的友元
}

int main() {
    B b;
    b.showA();
    return 0;
}
```

> 友元成员函数的优点是，只把私有成员真正给到真正需要访问的一个成员函数；封装破环小，权限最小原则；缺点是代码繁琐，需要先声明。


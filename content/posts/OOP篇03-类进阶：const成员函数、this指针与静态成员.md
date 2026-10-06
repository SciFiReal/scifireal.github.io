---
title: "OOP篇03-类进阶：const成员函数、this指针与静态成员"

date: 2026-10-06T17:30:46+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇03-类进阶：const成员函数、this指针与静态成员"

toc: true

---



## 1. const成员函数

在成员函数后面添加`const`

```
// 普通成员函数：可以读写成员变量
int getAge()
{
    age = 100; // ✅可以修改成员
    return age;
}

// const成员函数：承诺本函数不会修改任何成员变量
int getAge() const
{
    // age = 100; // ❌编译报错！不能修改成员
    return age;
}
```

> 原理：
>
> `const`成员函数，`this`指针变成`const Person* this`，也就是说当前对象被当成了常量对象，不能修改对象里面的数据。

> 为什么要这样用呢？
>
> 1. 当你不小心在查询函数中写了修改代码，编译会帮助你发现错误；
> 2. `const`对象只能调用`const`成员函数

```c++
const Person p; // 常量对象，不能修改里面数据
// p.setAge(18); // ❌普通成员函数，会修改数据，不能调用
p.getAge();     // ✅const成员函数，只读，可以调用
```

> 用在什么地方呢？
>
> 查询、打印、读取、计算，不改动成员的函数，全部加上末尾`const`

---

## 2. this指针

之前也提到过成员函数隐藏的`this`指针， 等价于`Person* this`,指向调用这个函数的对象。基础用法如下：

```c++
void setAge(int age)
{
    this->age = age; 
    // this->age：对象成员age；右边age：传入参数
}
```

> 有两个核心用途
>
> 1. 返回当前对象的引用，实现链式调用（作用：让你可以连续调用多个成员函数，写代码更连贯简洁，不用重复写对象名）

```c++
class Person{
private:
    int age;
public:
    Person& setAge(int a)
    {
        age = a;
        return *this; // 返回当前对象本身的引用
    }
};
// 链式调用
Person p;
p.setAge(10).setAge(20);
```

> 2. 在成员函数内部，可以拿到当前对象地址，用来和别的对象对比

---

## 3. 静态成员

静态成员包括静态成员变量和静态成员函数；静态成员不属于任何对象，是属于这个类，所有对象都共享这个内存。

1. 静态成员变量

```c++
class Person{
public:
    static int count; // 静态成员声明，属于类，不属于单个对象
    Person()
    {
        count++; // 每新建一个对象，总数+1
    }
};
// 静态变量！必须在类外面初始化！
int Person::count = 0;

int main()
{
    Person p1;
    Person p2;
    cout << Person::count; // 直接用类名访问，输出2
    return 0;
}
```

> 由上述例子可知：静态成员变量`static`，不管创建多少对象`p1`,`p2`，内存中只有这一份，`p1`和`p2`共享静态成员变量。

2. 静态成员函数

   特点是没有`this`指针，因为不属于任何对象。

```c++
class Person{
public:
    static int count;
    static void showCount()
    {
        cout << count; // ✅ 1. 可以访问静态变量
        // cout << age; // ❌不能访问普通成员，没有this，不知道是谁的age
    }
private:
    int age;
};
int Person::count = 0;

int main()
{
    Person::showCount(); // ✅2.不用创建对象，直接调用静态函数
    return 0;
}
```

> 由上述例子可知：
>
> 静态成员函数只能访问静态变量；
>
> 不用创建对象，直接调用静态函数；

> 怎么时候用呢？某个类共享数据/行为，不属于单个对象
>
> 1. 静态成员变量，保存类的全局信息，；
> 2. 静态成员函数，类的工具函数，不需要对象就能调用；
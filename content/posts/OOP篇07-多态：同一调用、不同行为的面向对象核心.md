---
title: "OOP篇07-多态：同一调用、不同行为的面向对象核心"

date: 2026-10-06T21:58:08+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇07-多态：同一调用、不同行为的面向对象核心"

toc: true
---





> 什么是多态？
>
> 同一个函数调用语句，传入不同对象，自动执行不同版本的函数代码。

> 为何需要多态？
>
> 1. 代码可以 “一套接口，适配多种子类”，新增子类不用修改原有业务代码
> 2. 统一接口，调用逻辑简单

---

## 1. 多态

```c++
#include <iostream>
#include <string>
using namespace std;

// 父类
class Person {
protected:
    string name;
public:
    Person(string n):name(n){}
    // ✅ 虚函数：为多态做准备
    virtual void work() {
        cout << name << "工作" << endl;
    }
};

// 学生：继承Person，重写work
class Student : public Person {
public:
    Student(string n):Person(n){}
    // 重写虚函数
    void work() override {
        cout << name << " 在学习" << endl;
    }
};

// 老师：继承Person，重写work
class Teacher : public Person {
public:
    Teacher(string n):Person(n){}
    void work() override {
        cout << name << " 在讲课" << endl;
    }
};

// 只写这一个通用函数！父类引用，支持所有子类
void doWork(Person& p) {
    p.work();
}

int main() {
    Student s("张三");
    Teacher t("李老师");

    doWork(s); // 同一个doWork，自动调用Student::work
    doWork(t); // 同一个doWork，自动调用Teacher::work
    return 0;
}
```

> 由上面例子可以知道多态生效的4个必要条件
>
> 1. 要有继承，子类继承父类；
> 2. 父类要有虚函数`virtual`；
> 3. 子类重写`override`这个虚函数（函数名，返回值，参数完全一致）
> 4. 使用父类指针/父类引用 指向子类对象
>
> 注意：如果不用父类指针/引用，直接`Person p = s`，会发生对象切片，多态失效

```c++
Student s("张三");
Person p = s; // 对象切片！只会拷贝父类部分，子类信息丢掉
p.work();     // 调用父类work，不是Student的work，多态失效
```

> 问题一：为什么可以写`Person& p = t`;

这是向上转型。允许父类引用 / 父类指针，**引用 / 指向子类对象**，不是把`t`变成 Person 类型；而是：**通过父类引用 p，只能以 Person 的视角去访问这个 t 对象**。

对象`t`真实类型：`Teacher`（永远不会变）；

引用`p`的静态类型：`Person`（编译器看这个类型）

静态类型：编译时就知道的类型（p 是 Person&）；

动态类型：运行时对象真实类型（t 是 Teacher）；

虚函数多态，就是靠**静态类型和动态类型不一致**实现的。

```c++
Teacher t("李老师");
Person& p = t;
p.work(); // 虚函数，运行时看真实对象是Teacher，调用Teacher::work

p.workId; // ❌ 编译报错！p是Person引用，编译器不知道有workId
```

编译时是Person引用，运行时是Teacher。



---

## 2. 虚析构函数

> 场景：`new` 创建子类对象，返回父类指针；最后 `delete` 父类指针。

情况 1：父类析构没有 virtual（错误写法！会内存泄漏）

```
#include <iostream>
#include <string>
using namespace std;

class Person {
protected:
    string name;
public:
    Person(string n):name(n){
        cout << "Person构造\n";
    }
    virtual void work() {
        cout << name << "工作\n";
    }
    // ❗父类析构没有 virtual
    ~Person() {
        cout << "Person析构\n";
    }
};

class Student : public Person {
private:
    char* buf; // 子类自己在堆上开辟内存
public:
    Student(string n, const char* str):Person(n){
        buf = new char[strlen(str)+1];
        strcpy(buf, str);
        cout << "Student构造\n";
    }
    void work() override {
        cout << name << "在学习：" << buf << "\n";
    }
    ~Student() {
        delete[] buf; // 释放子类堆内存！
        cout << "Student析构\n";
    }
};

int main() {
    // 父类指针接收new出来的子类对象
    Person* p = new Student("张三", "2026001");
    p->work();
    delete p; // ❗重点：父类析构不是virtual
    return 0;
}

```

输出结果：

```
Person构造
Student构造
张三在学习：2026001
Person析构
```

> 只调用了父类析构！Student 析构函数完全没跑！
>
> 子类里面 `buf` 是`new`出来的堆内存，`~Student()`没有执行 → `delete[] buf`不会执行 → **内存泄漏！**
>
> 原因：析构函数此时是**静态绑定**。编译器看指针的静态类型 `Person*`，直接调用`Person::~Person()`。不看真实对象类型。

情况 2：父类析构加上 virtual（正确写法）

```
只改动一行：virtual ~Person()
```

```
Person构造
Student构造
张三在学习：2026001
Student析构
Person析构
```

> 先子类析构，再父类析构。子类析构执行，`delete[] buf` 释放子类堆内存，没有内存泄漏。
>
> 原理：析构函数变成虚函数，触发动态绑定。`delete p` 的时候，**看对象真实类型是 Student，先调用 Student 析构，之后自动调用父类析构**。

> ==经验，如果一个类被设计成通用类，则写虚析构函数，后续可能会有虚函数而已==

---

## 3. 虚函数的原理

父类有虚函数，类内部会藏一个虚函数指针vptr，指向一张虚函数表。虚表里面存放当前对象类型对应的函数地址。运行时通过vptr查表，找到真实类型对应的函数，完成动态绑定。

虚表（vtable）里面存的是：本类所有虚函数的地址，包含虚析构函数的地址。

1. 先把两个对象内存画出来

```
Person对象内存：
┌──────────┬────────┐
│  vptr    │ name   │
└──────────┴────────┘
 ↑虚函数指针，指向 Person虚表(Person vtable)

Person虚表：存放Person虚函数地址
[ &Person::work , &Person::~Person ]
```

子类 Student : public Person（重写 work）

子类继承父类的 vptr，**子类自己会生成一张全新虚表**

```
Student对象内存：
┌──────────┬────────┬────────┐
│  vptr    │ name   │ id     │
└──────────┴────────┴────────┘
 ↑这个vptr指向 Student虚表(Student vtable)

Student虚表：
[ &Student::work , &Student::~Person ]
```

> 重点：**每个类（Person、Student）各有一张虚表，不是每个对象一张表！** 但是**每个对象都自带一个 vptr（虚指针），存自己所属类虚表的地址。**

2. `Person* p = new Student;` 内存情况

```
Person* p -----> 【Student对象内存】
                 ┌──────────┬────────┬────────┐
                 │  vptr    │ name   │ id     │
                 └──────────┴────────┴────────┘
                     ↓
                Student虚表
                [ &Student::work , ... ]
```

- 指针`p`的**静态类型是 Person***，编译器编译阶段只认 Person 成员。
- 但是`p`指向的那块内存，**真实对象是 Student，对象里面自带的 vptr 指向 Student 虚表**。

3. `p->work();` 运行时发生了什么（动态绑定）

   1. 拿到`p`指向对象内存里的**vptr**；
   2. 通过 vptr 找到**这个对象真实类型对应的虚表**；
   3. 在虚表里取出`work`函数地址；
   4. 调用这个地址的函数。

4. 如果对象是 Person 呢

   ```c++
   Person* p2 = new Person();
   // p2指向Person对象内存，里面vptr指向Person虚表。
   p2->work();  //查表拿到Person::work。
   ```

5. 如果没有 virtual（普通函数，静态绑定）

   `p->work()`，编译期直接看指针静态类型`Person*`，直接写死调用`Person::work`地址。**不去查表，完全不看对象真实类型**。

----

> 为什么需要虚表呢？

虚表是专门为==运行时动态选择函数==才造出来的。

编译器处理`p->work()`

1. 编译期检查：Person 有虚函数 work，语法合法；

2. **但是编译器不知道运行时 p 到底指向 Person 还是 Student**，所以不能写死函数地址；

3. 编译器生成的指令变成：

   > ① 从 p 指向对象内存取出 vptr 
   >
   > ② 通过 vptr 找到虚表
   >
   > ③ 在虚表取出 work 函数地址
   >
   > ④ 调用这个地址

虚表的作用：运行时根据对象真实类型，拿到正确的虚函数地址。没有动态选择的需求，完全不需要虚表！不管有没有虚函数，`p`（`Person* p = &s`）**永远指向子类对象 s 的起始地址**，指向的对象本身不会变。 **变的不是 p 指向谁，而是调用函数的绑定方式。**编译期看p的静态类型Person*，直接绑定 Person::work。

```c++
class Person {
public:
    void work() { cout << "Person work"; }
};
class Student : public Person {
public:
    void work() { cout << "Student work"; }
};
Student s;
Person* p = &s; // p仍然指向s（Student对象），对象本身是完整Student，没有变！
p->work();      // 编译期看p的静态类型Person*，直接绑定 Person::work
```

1. **有没有虚表，不改变指针指向哪个对象**；指针永远指向`s`这个 Student 对象。
2. **有没有虚表，决定调用函数的方式：**
   - 无虚表（普通函数）：编译期看指针静态类型，直接写死父类函数地址（静态绑定）。
   - 有虚表（虚函数）：运行时读取对象 vptr 查表，按**对象真实类型**选函数（动态绑定）。

---

## 4. 纯虚函数与抽象类

纯虚函数，就是没有实现的虚函数，虚表里面这个位置填 0。包含纯虚函数的类，叫抽象类，不能实例化（不能直接创建对象）。

1. 纯虚函数

```
virtual void work() = 0; // 纯虚函数，=0 是标记  
```

纯虚说明**这个函数在父类没有实现**，虚表中该项置空（0），支持多态

2. 抽象类规则

   只要类里面**至少有一个纯虚函数**，这个类就是抽象类。

   - **不能直接创建对象**（`Person p;` 编译报错）

   - 可以定义抽象类指针、引用（用来做多态）

   - 子类**必须重写全部纯虚函数**；如果子类不重写，子类也变成抽象类，同样不能实例化

```c++
#include <iostream>
#include <string>
using namespace std;

// 抽象类：包含纯虚函数 work
class Person {
protected:
    string name;
public:
    Person(string n):name(n){}
    // ✅纯虚函数，没有实现
    virtual void work() = 0;
    // 抽象类建议虚析构！父类指针delete子类要用
    virtual ~Person() {
        cout << "Person析构\n";
    }
};

// Student子类：重写纯虚函数，不再是抽象类，可以实例化
class Student : public Person {
public:
    Student(string n):Person(n){}
    void work() override {
        cout << name << " 在学习\n";
    }
};

// Teacher子类：重写纯虚函数
class Teacher : public Person {
public:
    Teacher(string n):Person(n){}
    void work() override {
        cout << name << " 在讲课\n";
    }
};

// 多态函数，接收抽象类引用
void doWork(Person& p) {
    p.work();
}

int main() {
    // Person p("张三"); // ❌编译报错！抽象类不能创建对象

    Student s("张三");
    Teacher t("李老师");
    doWork(s);
    doWork(t);

    Person* p1 = new Student("小明");
    p1->work();
    delete p1;
    return 0;
}
```

输出：

```
张三 在学习
李老师 在讲课
小明 在学习
Person析构
```


---
title: "OOP篇06-继承：从通用到特殊的代码复用机制"

date: 2026-10-06T20:13:32+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇06-继承：从通用到特殊的代码复用机制"

toc: true
---



> 什么是继承？
>
> 继承：让我们可以从一个通用类（基类/父类）中派生出特殊类（派生类/子类），自动获取基类所有的成员（代码复用），并在派生类中添加新的或修改旧的成员。
>
> 比如说Student is a Person。学生是人，学生类继承人类。子类和父类是is a的关系。

---

## 1. 继承的基本语法

```c++
#include <iostream>
#include <string>
using namespace std;

// 父类（基类）：通用的人
class Person {
protected:
    string name;
    int age;
public:
  	// 父类无参构造
    void showInfo() {
        cout << name << "，" << age << "岁" << endl;
    }
};

// 子类（派生类）Student，公有继承Person
class Student : public Person {
private:
    string id; // 学生独有：学号
public:
    void setStu(string n, int a, string i) {
        name = n;
        age = a;
        id = i;
    }
    void study() {
        cout << name << "正在学习" << endl;
    }
};

class Teacher : public Person {
private:
    string workId; // 老师独有：工号
public:
    void setTea(string n, int a, string w) {
        name = n;
        age = a;
        workId = w;
    }
    void teach() {
        cout << name << "正在讲课" << endl;
    }
};

int main() {
    Student s;
    s.setStu("张三",18,"2026001");
    s.showInfo(); // ✅ 继承父类的方法，直接使用
    s.study();

    Teacher t;
    t.setTea("李老师",35,"T001");
    t.showInfo();
    t.teach();
    return 0;
}
```

> 访问级别：
>
> 类：可以访问public/protected/private；
>
> 子类：可以访问public/protected；
>
> 外面：可以访问public；

> 三种继承方式：public/protected/private。
>
> public继承：基类public->public；基类protected->protected；基类private -> 不能访问
>
> protected继承：基类public->protected；基类protected->protected；基类private -> 不能访问
>
> private继承：基类public->private；基类protected->private；基类private -> 不能访问
>
> 实际中99%都使用public继承

---

## 2. 构造函数与析构函数在继承中的行为

创建子类对象：先调用父类构造函数，再调用子类构造函数。

销毁子类对象：先调用子类析构，再调用父类析构。

```c++
#include <iostream>
#include <string>
using namespace std;

// 父类（基类）：通用的人
class Person {
protected:
    string name;
    int age;
public:
    // 父类带参构造
    Person(string n, int a):name(n), age(a){}
    void showInfo() {
        cout << name << "，" << age << "岁" << endl;
    }
};

// 子类Student，公有继承Person
class Student : public Person {
private:
    string id; // 学生独有：学号
public:
    // 子类构造：初始化列表调用父类构造
    Student(string n, int a, string i):Person(n,a), id(i){}
    void study() {
        cout << name << "正在学习" << endl;
    }
};

class Teacher : public Person {
private:
    string workId; // 老师独有：工号
public:
    Teacher(string n, int a, string w):Person(n,a), workId(w){}
    void teach() {
        cout << name << "正在讲课" << endl;
    }
};

int main() {
    Student s("张三",18,"2026001");
    s.showInfo();
    s.study();

    Teacher t("李老师",35,"T001");
    t.showInfo();
    t.teach();
    return 0;
}
```


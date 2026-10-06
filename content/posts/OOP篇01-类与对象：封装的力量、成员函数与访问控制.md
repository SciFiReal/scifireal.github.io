---
title: "OOP篇01-类与对象：封装的力量、成员函数与访问控制"

date: 2026-10-06T09:58:55+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇01-类与对象：封装的力量、成员函数与访问控制"

toc: true
---



## 1. 类的封装

之前我们学过结构体是把数据打包，函数是工具打包，类是把数据和操作数据的函数一起打包，这就是类的封装。在类中叫成员变量和成员函数。

使用封装的理由：

- 隐藏内部细节，不能让外部随意更改成员数据；
- 修改内部代码，提供外部接口不变，外面代码不用变；
- 代码易维护；

---

## 2. 访问控制

控制类外面的代码能不能访问这个成员。一共有三种形式：

- `public`：公开，外部直接访问，接口；
- `private`：私有，只有类的成员函数能访问，类的外面不能访问
- `protected`：外部不能访问，子类可以访问，一般在继承时用到

> `class`默认是`private`，`struct(结构体)` 默认是 `public`

```c++
class Person
{
private:
  // 私有成员：外面不能直接修改，保护数据
  int age;
public:
  // 公有成员函数：对外接口，用来读写私有age
  void setAge(int a)
  {
    // 在函数里可以加校验，防止非法数据
    if(a > 0 && a < 150)
    {
      age = a;
    }
    else
    {
      age = 0;
    }
  }
  int getAge()
  {
    return age;
  }
};
```

```c++
int main()
{
  Person p;
  // p.age = 20; // ❌报错！age是private，main属于类外面，不能直接访问
  p.setAge(20);  // ✅ 调用公有函数修改age，函数内部做校验
  std::cout << p.getAge();
  return 0;
}
```

---

## 3. 成员函数的两种定义方式

成员函数是属于这个类的函数，可以访问这个类中所有成员变量。

1. 类里直接写（内联），代码短时比较方便

   ```c++
   class Person
   {
   private:
     int age;
   public:
     // 在类内部定义成员函数
     void setAge(int a)
     {
       if(a>0 && a<150) age = a;
     }
     int getAge()
     {
       return age;
     }
   };
   ```

2. 类内声明，类外面定义（开发常用）

   > 和函数一样，头文件写类的声明，cpp中实现函数细节；
   >
   > 类外面要使用`::`作用域解析符来表示这个函数是属于这个类的；

   ```c++
   class Person
   {
   private:
     int age;
   public:
     void setAge(int a);  // 只声明，不写实现
     int getAge();        // 只声明
   };
   
   // 类外面实现成员函数，必须加 Person::
   void Person::setAge(int a)
   {
     if(a>0 && a<150)
     {
       age = a;
     }
   }
   
   int Person::getAge()
   {
     return age;
   }
   ```

   ---

## 4. 成员函数隐藏指针

```c++
class Person
{
private:
  int age;
public:
  void setAge(int a); // 类内声明
  void getAge();  
};

// 类外实现
void Person::setAge(int age) { 
  this->age = age; 
}

void Person::getAge() {
	std::cout << this->age << "\n";
}
```

```c++
int main()
{
  Person p;      // 在栈上创建一个Person对象p
  p.setAge(18);  // 调用setAge
  p.getAge();   // 18
  return 0;
}
```

> `this`是调用这个函数的对象的地址，指向调用这个函数的对象，也就是说`this = &p`
>
> 之前我的疑惑是：这个 `private` 不能不能随便被访问吗，`private` 限制的是类外部面的代码；成员函数是属于类的内部代码，对象Person可以访问`private `。

---

> 还有一个小知识点：与之前类似普通结构体用`.`访问；结构体指针用`->`访问；
>
> 对象的访问：栈对象用 `.` 访问成员，指针对象用 `->` 访问成员。

```c++
Person p;   // p 是栈上实实在在的对象（实体）
p.setAge(18); // 对象.成员

Person* ptr = &p; // ptr是指针变量，只存对象p的地址，不是对象本身
ptr->setAge(20);  // 指针->成员   ptr-> = *ptr.
```


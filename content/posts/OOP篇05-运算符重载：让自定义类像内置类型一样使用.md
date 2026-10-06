---
title: "OOP篇05-运算符重载：让自定义类像内置类型一样使用"

date: 2026-10-06T19:17:33+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "OOP篇05-运算符重载：让自定义类像内置类型一样使用"

toc: true
---



> 运算符重载是什么？

运算符重载，就是给`+ - * / << >> ==`这些符号，重新定义一套属于我们自己类的逻辑；但符号本身的优先级、结合性、操作个数不能改。比如`int a=1; b=2; a+b`，编译器知道是整数相加。自定义类（比如坐标、复数、时间、学生）默认不能直接写`obj1+obj2`，编译器不知道`+`代表什么。运算符重载就是解决这个问题。

> 为什么需要运算符重载呢？

1. 写一个复数类，实现两个复数相加，不能重载运算符，就要写成员函数`c1.add(c2)`

   ```c++
   Complex c1(1,2), c2(3,4);
   Complex res = c1.add(c2); // 调用函数，语义弱
   ```

   重载`+`之后；

   ```c++
   Complex res = c1 + c2;   // 一眼就能看懂
   ```

2. 复用已有语法习惯，降低理解成本

   大家写代码，都能懂`+ - * <<`等运算符。不用去记忆一堆自定义函数名`add() print()`。

3. 方便对接标准库（比如cout输出）

   重载`<<`，才能直接用标准输出流打印对象，而不是每次调用`obj.print()`。

> 两种实现方式：成员函数重载；全局（友元）函数重载

---

## 1. 成员函数重载

```c++
class Complex{
private:
    double real; //实部
    double imag; //虚部
public:
    Complex(double r,double i):real(r),imag(i){}
    //成员函数重载 +
    Complex operator+(const Complex& other){
        return Complex(real + other.real, imag + other.imag);
    }
};
```

```c++
Complex c1(1,2), c2(3,4);   // 使用
Complex c3 = c1 + c2;
```

- 成员函数重载隐藏了第一个参数：`this`（左操作数），上述例子`(this).real`

- `c1 + c2` 等价于 `c1.operator(c2)`
- 适用场景：运算符左边是自定义类对象，不适合`cout << c1`，左边是`ostream`，不是自定义对象

---

## 2. 全局友元函数

```c++
#include <iostream>
using namespace std;

class Complex{ 
private:    
    double real;     // 实部，私有成员
    double imag;     // 虚部，私有成员
public:    
    Complex(double r,double i):real(r),imag(i){}  // 构造函数
    friend ostream& operator<<(ostream& os, const Complex& c); 
    // 友元声明：全局函数 operator<< 是本类的友元
};

// 全局函数定义：重载 <<
ostream& operator<<(ostream& os, const Complex& c){    
    os << c.real << "+" << c.imag << "i";    
    return os; 
}

int main()
{
    Complex c(1,2);
    cout << c;  // 编译器自动翻译成 operator<<(cout, c);  1+2i
    return 0;
}
```

> `friend ostream& operator<<(ostream& os, const Complex& c); `
>
> - `friend`：告诉 Complex 类：这个全局函数是我的朋友，可以访问我的 private。
>
> - `operator<<`：函数名，C++ 运算符重载固定写法，代表重载 `<<`。
>
> - 返回值：`ostream&` 流的引用。
>
> - 参数 1：`ostream& os` —— 就是 cout（输出流对象，引用传递，避免拷贝）
>
> - 参数 2：`const Complex& c` —— 我们要打印的复数对象，const 引用，不拷贝、不修改对象

> `// 全局函数定义：重载 <<
> ostream& operator<<(ostream& os, const Complex& c){    
>     os << c.real << "+" << c.imag << "i";    
>     return os; 
> }`
>
> - `os << c.real`：往输出流 os 里面写入 c 的实部。因为是友元，**能直接拿 private 的 real 和 imag**
> - 拼接字符串，输出类似 `1+2i`。
> - `return os;` 返回流对象引用。

> 为什么要返回 os（ostream&）？
>
> 支持连续输出！例如cout << "复数是：" << c << endl；`cout << "复数是："` 返回 cout； 然后用返回的 cout 继续执行 `cout << c`； 再返回 cout，执行 `cout << endl;`

---

> 为什么需要全局友元函数呢？

```c++
class Complex{
private:
    double real;
    double imag;
public:
    Complex(double r,double i):real(r),imag(i){}
    void print() const{
        cout << real << "+" << imag << "i";
    }
};
---
Complex c(1,2);
c.print();
```

缺点是==不能链式输出==，不能写成 `cout << "复数：" << c << endl;`，必须拆成多行

```c++
cout << "复数：";
c.print();
cout << endl;
```

---

## 3. 不能重载的运算符

以下运算符不能重载：

- `.`（成员访问）
- `.*`（成员指针访问）
- `::`（作用域解析）
- `?:`（三目运算符）
- `sizeof`
- `typeid`
- 以及少数其他特殊运算符

**可以重载但应谨慎的**：`&&`、`||`（短路求值会失效）、`,`（逗号运算符重载后很迷惑）、`&`（取地址运算符重载会混淆）

---

## 4. 常用的运算符重载

1. 赋值运算符`operator=`
2. 下标运算符 `operator[]`
3. 自增/自减运算符`operator++/operator--）`

---

==我建议是尽量避免使用运算符重载==

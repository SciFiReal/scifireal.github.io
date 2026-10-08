---
title: "高级篇01-异常处理：try/catch/throw与RAII异常安全"

date: 2026-10-07T08:20:26+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇01-异常处理：try/catch/throw与RAII异常安全"

toc: true
---



## 1. 异常的基本语法

```c++
#include <iostream>
#include <stdexcept>  // C++标准异常库头文件

// 取款函数：业务逻辑出错时，主动 throw 异常
double withdraw(double balance, double amount)
{
    // 业务规则1：取款金额必须为正数
    if (amount <= 0)
    {
        // 1. throw：抛出一个异常对象，立刻中断当前函数，不再往下执行
        // invalid_argument 是标准异常类型，表示参数非法
        throw std::invalid_argument("取款金额必须大于0");
    }

    // 业务规则2：余额不足
    if (amount > balance)
    {
        // runtime_error 是标准异常类型，表示运行时业务错误
        throw std::runtime_error("余额不足，当前余额：" + std::to_string(balance));
    }

    // 正常逻辑：所有校验都通过，才会走到这里执行扣款
    balance -= amount;
    std::cout << "取款成功，剩余余额：" << balance << std::endl;
    return balance;
}

int main()
{
    double my_balance = 1000.0;

    // 2. try：包裹「可能抛出异常」的代码块
    try
    {
        std::cout << "=== 开始办理取款 ===" << std::endl;
        withdraw(my_balance, 1500.0);  // 余额1000，尝试取1500
        std::cout << "=== 交易完成 ===" << std::endl;
    }
    // catch：按类型捕获异常，对应不同错误做不同处理
    catch (const std::invalid_argument& e)
    {
        std::cout << "参数错误：" << e.what() << std::endl;
    }
    catch (const std::runtime_error& e)
    {
        std::cout << "交易失败：" << e.what() << std::endl;
    }

    std::cout << "\n程序继续运行，没有崩溃" << std::endl;
    return 0;
}
```

输出结果：

```c
=== 开始办理取款 ===
交易失败：余额不足，当前余额：1000.000000

程序继续运行，没有崩溃
```

异常执行流程：

1. 程序进入`try`模块，按顺序执行正常业务代码；
2. 调用`withdraw`函数，内部检测到余额不足，执行`throw`；
3. `throw`会立即中断当前函数，后面的扣款代码不再执行，异常对象沿着调用栈向上抛出；
4. 外层匹配到对应类型的`catch`，进入catch执行错误处理；
5. 处理完后程序继续往下走，不会崩溃；

---

## 2. 异常类型：可以抛出任何类型

```
std::exception  【所有标准异常的根基类，提供虚函数 what()】
├─ std::logic_error  【逻辑错误：代码逻辑缺陷导致，理论上可通过正确编码避免】
│   ├─ std::invalid_argument    无效参数
│   ├─ std::out_of_range        下标/范围越界
│   ├─ std::domain_error        定义域错误（数学函数输入非法）
│   └─ std::length_error        长度超出最大限制
│
├─ std::runtime_error  【运行时错误：运行环境导致，无法完全预判避免】
│   ├─ std::overflow_error      算术上溢
│   ├─ std::underflow_error     算术下溢
│   ├─ std::range_error         结果值范围溢出
│   └─ std::system_error        系统级错误（文件/网络/操作系统错误）
│
└─ 其他直接派生异常（底层系统级错误）
    ├─ std::bad_alloc           new 内存分配失败
    ├─ std::bad_cast            dynamic_cast 转型失败
    ├─ std::bad_typeid          typeid 操作空指针
    └─ std::bad_exception       异常规格不匹配
```

推荐：**继承标准异常类**，这样可以统一用 `catch (const exception& e)` 捕获。

---

## 3. RAII与异常安全：保证资源释放

异常的最大挑战：异常发生时，函数提前返回，普通的delete/close/fclose等清理代码可能永远不会执行。

错误例子：

```c++
bool processData() {
    mtx.lock();                   // 1. 加锁
    FILE* fp = fopen("data.txt", "r"); // 2. 打开文件
    if (!fp) {
        mtx.unlock();             // ❌ 必须手动释放锁，漏写就死锁
        return false;
    }

    char* buf = new char[1024];   // 3. 分配缓冲区
    if (/* 读取数据失败 */) {
        delete[] buf;             // ❌ 必须按顺序释放所有资源
        fclose(fp);
        mtx.unlock();
        return false;	
    }

    // ... 正常处理数据 ...

    // 正常返回也要释放一遍
    delete[] buf;
    fclose(fp);
    mtx.unlock();
    return true;
}

```

RAII指针+异常

RAII（Resource Acquisition Is Initialization，资源获取即初始化）是 C++ 的核心资源管理范式：

- **构造时获取资源**：对象构造的时候，申请 / 打开资源（内存、文件、锁、网络连接等）；
- **析构时释放资源**：对象生命周期结束时，析构函数自动释放对应的资源；
- 把资源的生命周期和对象的生命周期**完全绑定**。

常见的标准库 RAII 类：

- 内存：`std::string`、`std::vector`、`std::unique_ptr`、`std::shared_ptr`
- 文件：`std::fstream`
- 锁：`std::lock_guard`、`std::unique_lock`
- 其他：套接字、数据库连接句柄等

```c++
#include <fstream>
#include <mutex>
#include <vector>

void processData() {
    std::lock_guard<std::mutex> lock(mtx); // 1. RAII锁：出作用域自动解锁
    std::ifstream file("data.txt");        // 2. RAII文件：出作用域自动关闭
    if (!file.is_open()) {
        throw std::runtime_error("打开文件失败");  
    }

    std::vector<char> buf(1024);           // 3. RAII内存：出作用域自动释放

    if (/* 读取数据失败 */) {
        throw std::runtime_error("读取数据失败");
    }

    // ... 正常处理数据 ...

    // 不用写任何释放代码！
}

int main() {
    try {
        processData();
    } catch (const std::exception& e) {
        std::cout << "处理失败：" << e.what() << std::endl;
    }
    // 无论哪里抛异常，锁、文件、内存都已经被自动释放了
}
```

- 每个资源只需要声明一次，释放逻辑自动生效；
- 无论从哪一行抛异常，栈展开都会自动调用所有局部对象的析构函数；
- 代码里全是业务逻辑，没有冗余的释放代码，可读性和可维护性碾压错误码写法。

> **后续要学的标准库智能指针**（unique_ptr/shared_ptr）本质上就是RAII类——它们在构造时接管指针，在析构时自动delete。

---

上述例子就是栈展开和RAII指针的结合使用

**栈展开：是异常抛出时自动销毁栈上局部变量的行为（机制）** 

**RAII：把释放资源的代码写进对象析构函数（设计手法）** 

栈展开负责**调用析构**，RAII 负责**析构里释放资源**。

> 上述例子中：打开文件失败，抛出异常

```c++
std::lock_guard<std::mutex> lock(mtx);  // ✅ 构造完成
std::ifstream file("data.txt");         // ✅ 构造完成
if (!file.is_open()) {
    throw std::runtime_error("打开文件失败"); // 这里throw！触发栈展开
}
```

throw之后，栈展开启动：

1. 在`processData`函数作用域内，逆序销毁已经构造成功的局部对象
   - 先销毁 `file`（ifstream，析构自动 close 文件）
   - 再销毁 `lock`（lock_guard，析构自动 unlock 互斥锁）
2. 然后退出`processData`，回到 main 里面 try 块，匹配到 catch 捕获异常。

> 上述例子中：文件打开成功，读到后面抛异常

```c++
std::lock_guard<std::mutex> lock(mtx);
std::ifstream file("data.txt");
if (!file.is_open()) {
    throw ...;
}
std::vector<char> buf(1024); // ✅ buf构造完成
if (/*读取失败*/) {
    throw std::runtime_error("读取数据失败"); // throw触发栈展开
}
```

throw → **栈展开开始**，逆序析构当前作用域已经构造好的局部变量：

1. 销毁 `buf` → vector 析构，释放堆内存
2. 销毁 `file` → 关闭文件
3. 销毁 `lock` → 解锁 mutex
4. 离开 `rocessData`，跳到 main 的 catch 块。

三个 RAII 对象，**栈展开自动调用它们的析构函数**。 RAII 保证析构函数里面做资源释放；栈展开保证**异常跑路时析构一定会被执行**。

---

## 4. noexcept

可以声明一个函数**不会**抛出异常

```c++
void simpleFunc(int x) noexcept// 承诺：这个函数绝不会抛异常
{
    // 内部如果抛了异常，程序直接terminate（不会传播到外部）
}
```

优势：

- 给调用者承诺：调用这个函数不需要担心异常；
- 给编译器优化信号：noexcept的函数可以更高效；
- 移动构造/移动赋值通常声明为noexcept（后续学习）；
- 旧版异常规范（throw()）已弃用，C++11以后统一用noexcept；

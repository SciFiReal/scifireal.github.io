---
title: "高级篇04-STL容器基础：vector、list、deque"

date: 2026-10-08T14:36:08+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇04-STL容器基础：vector、list、deque"

toc: true
---



> 我们之前学过的基础篇的动态内存管理，需要用new手动分配数组，但每次都要自己管理大小、扩容、释放。STL（Standard Template Library—标准模板库）把这种工作自动化。它提供一套==模板化的容器类==，用泛型编程的方式管理任意类型的数据，内部自动处理内存分配和释放。

> 什么是STL？

STL是C++标准库的核心组件之一，包含四大部分：

- ==容器==：存储数据的结构（vector，list，map，set等）；
- ==算法==：操作容器数据的通用算法（sort，find，copy等）；
- ==迭代器==：连接容器和算法的桥梁（像指针一样遍历容器）；
- 函数对象/==适配器==：灵活扩展算法行为；

STL的核心设计理念是模板化+泛型编程：写一次模板代码，可以用在任意类型上（如`vector<int>`存int，`vector<string>`存字符串，`vector<Device>`存自定义类）。

接下来介绍三大序列容器：

- `vector`（向量/动态数组）：连续内存，支持O(1)随机访问，我i不插入O(1)，中间插入O(n)
- `list`（双向链表）：节点不连续，任何位置插入/删除都是O(1)，但随机访问要O(n)
- `deque`(双端队列)：分段连续内存，头尾插入/删除都是O(1)，支持随机访问O(1)

## 1. vector：最常用的动态数组

vector在内部维护一块连续内存，像数组一样用下标访问，但可以自动扩容。

```c++
#include <iostream>
#include <vector>   // vector 动态数组头文件
#include <string>
#include <stdexcept> // out_of_range异常需要这个头文件
using namespace std;

// 传感器数据结构体：存放传感器编号、名称、温度
struct SensorData
{
    int id;
    string name;
    double temperature;

    // 构造函数：创建对象时直接赋值
    SensorData(int i = 0, string n = "", double t = 0)
        : id(i), name(n), temperature(t)
    {}
};

int main()
{
    // ===================== 1. vector 初始化 =====================
    cout << "=== 1. vector 初始化 ===" << endl;
    vector<int> v1;                  // 空vector：存放int，当前没有元素
    vector<int> v2(10, 0);           // 创建10个int元素，全部初始化为0
    vector<int> v3 = {1, 2, 3, 4, 5};// 列表初始化，直接放入5个数字
    vector<string> vs(3, "Hello");   // 创建3个字符串，都是"Hello"

    cout << "v1当前元素个数(size): " << v1.size() << endl;
    cout << "v3里面的元素: ";
    for (size_t i = 0; i < v3.size(); i++)
        cout << v3[i] << " ";
    cout << endl;

    // ===================== 2. push_back：尾部添加元素 =====================
    cout << "\n=== 2. push_back 在尾部插入元素 ===" << endl;
    vector<SensorData> sensors;
    // 在数组末尾追加传感器对象
    sensors.push_back(SensorData(1001, "加热炉A", 78.5));
    sensors.push_back(SensorData(1002, "冷却塔B", 28.3));
    sensors.push_back(SensorData(2001, "反应釜C", 125.8));
    cout << "插入3个传感器后，元素数量: " << sensors.size() << endl;

    // ===================== 3. 访问vector元素 =====================
    cout << "\n=== 3. 访问vector元素 ===" << endl;
    // []下标访问：速度快，但越界不会报错（不安全）
    cout << "sensors[0]: ID=" << sensors[0].id
         << " " << sensors[0].name
         << " 温度=" << sensors[0].temperature << "度" << endl;

    // at()访问：越界会抛出异常，安全访问方式
    try
    {
        cout << "尝试访问 sensors.at(10): " << sensors.at(10).name << endl;
    }
    catch (const out_of_range& e)
    {
        cout << "at(10) 越界错误: " << e.what() << endl;
    }

    // front() 获取第一个元素；back() 获取最后一个元素
    cout << "第一个传感器: " << sensors.front().name << endl;
    cout << "最后一个传感器: " << sensors.back().name << endl;

    // ===================== 4. vector 遍历方式 =====================
    cout << "\n=== 4. vector 的两种遍历方式 ===" << endl;
    cout << "方式1：下标索引遍历（简单直观）" << endl;
    for (size_t i = 0; i < sensors.size(); i++)
    {
        cout << "  [" << i << "] ID=" << sensors[i].id
             << " " << sensors[i].name
             << " 温度=" << sensors[i].temperature << "度" << endl;
    }

    cout << "方式2：迭代器遍历（STL标准写法）" << endl;
    // begin() 指向第一个元素；end() 指向末尾空白位置（不是最后一个元素）
  	// iterator（迭代器）就是 STL 容器的 “通用指针”，用来遍历容器里的元素
    for (vector<SensorData>::iterator it = sensors.begin();
         it != sensors.end();
         ++it)
    {
        cout << "  ID=" << it->id << " " << it->name << endl;
    }

    // ===================== 5. 删除元素 =====================
    cout << "\n=== 5. 删除元素 ===" << endl;
    sensors.pop_back();  // pop_back：删除末尾元素，不返回被删的值
    cout << "pop_back删除末尾元素后，数量: " << sensors.size() << endl;

    sensors.clear();     // clear：清空vector中所有元素，size变成0
    cout << "clear清空后 size=" << sensors.size()
         << " 是否为空: " << (sensors.empty() ? "是" : "否") << endl;

    // ===================== 6. size 和 capacity：元素数量 vs 内存容量 =====================
    cout << "\n=== 6. 容量管理 size / capacity / reserve / resize ===" << endl;
    vector<int> data;
    //capacity()；底层已经分配好的内存容量
    cout << "初始状态: size=" << data.size() << " capacity=" << data.capacity() << endl;

    // reserve：预留内存空间！只开内存，**不会创建元素，size不变**
    data.reserve(1000);
    cout << "reserve(1000)预留空间后: size=" << data.size()
         << " capacity=" << data.capacity() << endl;

    for (int i = 0; i < 500; i++)
        data.push_back(i);
    cout << "插入500个元素后: size=" << data.size()
         << " capacity=" << data.capacity() << endl;

    // resize：修改元素个数！会新增/销毁元素，改变size
    data.resize(100);
    cout << "resize(100)调整元素数量后: size=" << data.size()
         << " capacity=" << data.capacity() << endl;

    // ===================== 7. 嵌套vector：模拟二维数组 =====================
    cout << "\n=== 7. 嵌套vector（二维动态数组） ===" << endl;
    // 3行4列，全部初始化为0
    vector<vector<int>> matrix(3, vector<int>(4, 0));
    matrix[0][0] = 1;
    matrix[1][2] = 5;
    matrix[2][3] = 9;

    for (size_t row = 0; row < matrix.size(); row++)
    {
        for (size_t col = 0; col < matrix[row].size(); col++)
        {
            cout << matrix[row][col] << " ";
        }
        cout << endl;
    }

    return 0;
}
```

输出结果：

```
=== 1. vector 初始化 ===
v1当前元素个数(size): 0
v3里面的元素: 1 2 3 4 5

=== 2. push_back 在尾部插入元素 ===
插入3个传感器后，元素数量: 3

=== 3. 访问vector元素 ===
sensors[0]: ID=1001 加热炉A 温度=78.5度
at(10) 越界错误: invalid vector subscript
第一个传感器: 加热炉A
最后一个传感器: 反应釜C

=== 4. vector 的两种遍历方式 ===
方式1：下标索引遍历（简单直观）
  [0] ID=1001 加热炉A 温度=78.5度
  [1] ID=1002 冷却塔B 温度=28.3度
  [2] ID=2001 反应釜C 温度=125.8度
方式2：迭代器遍历（STL标准写法）
  ID=1001 加热炉A
  ID=1002 冷却塔B
  ID=2001 反应釜C

=== 5. 删除元素 ===
pop_back删除末尾元素后，数量: 2
clear清空后 size=0 是否为空: 是

=== 6. 容量管理 size / capacity / reserve / resize ===
初始状态: size=0 capacity=0
reserve(1000)预留空间后: size=0 capacity=1000
插入500个元素后: size=500 capacity=1000
resize(100)调整元素数量后: size=100 capacity=1000

=== 7. 嵌套vector（二维动态数组） ===
1 0 0 0
0 0 5 0
0 0 0 9
```

---

## 2. list：双向链表

`list`由节点组成，每个节点包含数据和两个指针（指向前一个和后一个节点）。

```c++
#include <iostream>
#include <list>   // STL链表容器头文件
#include <string>
using namespace std;

// 任务结构体：任务编号、任务名称、优先级
struct Task
{
    int id;
    string name;
    int priority;
    Task(int i, string n, int p) : id(i), name(n), priority(p) {}
};

int main()
{
    // list：双向链表容器
    list<Task> tasks;

    // ========== 1. 添加元素 ==========
    // push_back：尾部插入；push_front：头部插入，双向链表头尾插入都是O(1)极快
    tasks.push_back(Task(1, "数据采集", 2));
    tasks.push_back(Task(2, "数据处理", 1));
    tasks.push_back(Task(3, "数据存储", 3));
    tasks.push_front(Task(0, "初始化", 1)); // 在链表最前面插入

    cout << "任务列表（共" << tasks.size() << "个）:" << endl;
    // list只能用迭代器遍历，没有下标[]
    for (list<Task>::iterator it = tasks.begin(); it != tasks.end(); ++it)
    {
        cout << "  ID=" << it->id << " [" << it->priority << "] " << it->name << endl;
    }

    // ========== 2. 获取首尾元素 ==========
    cout << "\n第一个任务: " << tasks.front().name << endl;
    cout << "最后一个任务: " << tasks.back().name << endl;

    // ========== 重点！list不支持 [] 下标随机访问 ==========
    // tasks[1];  // ❌ 编译报错！
    // 底层是分散的链表节点，内存不连续，不能直接跳转到第N个元素

    // ========== 3. 在任意位置插入元素 ==========
    list<Task>::iterator it = tasks.begin();
    ++it;  // 迭代器向后移动，指向第二个任务
    tasks.insert(it, Task(99, "紧急任务", 0)); // 在it指向的位置前面插入新任务

    cout << "\n插入紧急任务后:" << endl;
    for (list<Task>::iterator it2 = tasks.begin(); it2 != tasks.end(); ++it2)
        cout << "  " << it2->name << " (优先级" << it2->priority << ")" << endl;

    // ========== 4. 删除元素 ==========
    tasks.pop_front();  // 删除头部元素
    tasks.pop_back();   // 删除尾部元素
    cout << "\n删除首尾后剩 " << tasks.size() << " 个任务" << endl;

    // ========== 5. list 独有功能：merge合并、reverse反转 ==========
    list<int> list1, list2;
    list1.push_back(1); list1.push_back(3); list1.push_back(5);
    list2.push_back(2); list2.push_back(4); list2.push_back(6);

    list1.merge(list2); // 合并两个【已经有序】的链表，合并后list2变成空链表
    cout << "\n合并后:";
    for (list<int>::iterator it3 = list1.begin(); it3 != list1.end(); ++it3)
        cout << " " << *it3;
    cout << endl;

    list1.reverse(); // 链表反转
    cout << "反转后:";
    for (list<int>::iterator it4 = list1.begin(); it4 != list1.end(); ++it4)
        cout << " " << *it4;
    cout << endl;

    return 0;
}
```

输出结果：

```
任务列表（共4个）:
  ID=0 [1] 初始化
  ID=1 [2] 数据采集
  ID=2 [1] 数据处理
  ID=3 [3] 数据存储

第一个任务: 初始化
最后一个任务: 数据存储

插入紧急任务后:
  初始化 (优先级1)
  紧急任务 (优先级0)
  数据采集 (优先级2)
  数据处理 (优先级1)
  数据存储 (优先级3)

删除首尾后剩 3 个任务

合并后: 1 2 3 4 5 6
反转后: 6 5 4 3 2 1
```

> 疑问：`list.merge()`为何能把两个有序列表合并后变成另外一个有序列表呢，如果是无序呢?

上述示例如下：

```
list1：1 → 3 → 5   （从小到大，本身有序）
list2：2 → 4 → 6   （从小到大，本身有序）
```

1. 拿两个迭代器，分别指向 list1 头部、list2 头部；
2. 比较两个迭代器的值，**把更小的那个节点摘下来，接到 list1 的尾部；**
3. 不断重复，直到其中一个链表取光；
4. 最后把剩下没取完的链表直接接在后面；
5. 合并完成后，**list2 会变成空链表（节点全部转移走了，不是拷贝！）；**

简单来说，1和2比较，1小，取1；3和2比较，2小，取2；3和4比较，取3；5和4比较，4小，取4；5和6比较，5小，取6；最后取6。同样无序列表合并也是这个逻辑。

---

## 3. deque：双端队列

`deque`（Double-Ended QUEue）是vector和list的折中方案：分段连续内存，支持随机访问，头尾插入都很快。

```c++
#include <iostream>
#include <deque>   // deque 双端队列头文件
#include <string>
using namespace std;

int main()
{
    // deque：double-ended queue，双端队列
    deque<string> msgQueue;  // 消息队列容器，可以在头部、尾部快速增删

    // ========== 1. 头部、尾部都可以高效插入元素 ==========
    msgQueue.push_back("消息1: 正常数据");        // 在【尾部】插入
    msgQueue.push_back("消息2: 正常数据");
    msgQueue.push_front("优先级消息: 紧急报警");  // 在【头部】插入，放高优先级消息
    msgQueue.push_front("优先级消息: 系统重启");

    // ========== 2. 支持随机访问，像vector一样用 [] 下标访问 ==========
    cout << "队列大小: " << msgQueue.size() << endl;
    for (size_t i = 0; i < msgQueue.size(); i++)
        cout << "  [" << i << "] " << msgQueue[i] << endl;

    // ========== 3. 头部、尾部都可以高效删除元素 ==========
    cout << "\n取出队首消息: " << msgQueue.front() << endl;
    msgQueue.pop_front();  // 删除头部元素
    cout << "取出队尾消息: " << msgQueue.back() << endl;
    msgQueue.pop_back();   // 删除尾部元素

    cout << "\n剩余 " << msgQueue.size() << " 条消息:" << endl;
    for (size_t i = 0; i < msgQueue.size(); i++)
        cout << "  " << msgQueue[i] << endl;

    return 0;
}
```

输出结果：

```
队列大小: 4
  [0] 优先级消息: 系统重启
  [1] 优先级消息: 紧急报警
  [2] 消息1: 正常数据
  [3] 消息2: 正常数据

取出队首消息: 优先级消息: 系统重启
取出队尾消息: 消息2: 正常数据

剩余 2 条消息:
  优先级消息: 紧急报警
  消息1: 正常数据
```

---

> 选择建议：

- 大多数场景用vector——随机访问块，默认首选；
- 需要两端都频繁插入，删除(如队列，任务池)，用deque；
- 需要两端频繁插入/删除且不需要随机访问，用list

---

## 4. 探究deque的底层实现方式

> 疑问：为什么`deque`的访问时间复杂度是O(1)？

> deque = **中控索引数组 + 多个独立、等长的内存块** 。
>
> 每一块内部是连续数组；块与块之间**不连续**。 
>
> 中控表记录每一个内存块的起始地址，用来快速定位。

假设设定：每个内存块最多放 4 个元素。

```
中控表 map（存放各个内存块的起始地址，是数组）
┌──────┬──────┬──────┐
│块0地址│块1地址│块2地址│
└──────┴──────┴──────┘
    ↓        ↓        ↓
┌────┬────┬────┬────┐  ┌────┬────┬────┬────┐  ┌────┬────┬────┬────┐
│ 0  │ 1  │ 2  │ 3  │  │ 4  │ 5  │ 6  │ 7  │  │ 8  │ 9  │空 │空 │
└────┴────┴────┴────┘  └────┴────┴────┴────┘  └────┴────┴────┴────┘
   块0                    块1                   块2
```

块内部内存连续；块 0、块 1、块 2 在内存里互相不挨在一起。

举例：访问 deque [5]。

元素下标 5，每个块容量 = 4

1. 块编号：`5 / 4 = 1` → 去中控表拿到「块 1 的起始地址」
2. 块内位置：`5 % 4 = 1` → 在块 1 里面取第 1 号位置的值（数字 5）

两步算术，不管 deque 有 1000 个元素，算法步骤不变 → **O(1)**
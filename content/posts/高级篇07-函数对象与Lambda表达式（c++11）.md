---
title: "高级篇07-函数对象与Lambda表达式（c++11）"

date: 2026-10-09T09:40:22+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇07-函数对象与Lambda表达式（c++11）"

toc: true
---



> 在STL算法中，我们频繁把函数名作为参数传给算个，比如sort的第三个参数是比较函数、find_if的第三个参数是条件判断函数、for_each的第三个参数是操作函数。这种能被当作函数调用的东西就是**函数对象**。C++新增的Lambda表达式让创建函数对象变得极其简单。

> 什么是函数对象？

C++函数对象 = 重载`operator()`的类/结构体的实例，本质是一个对象，但语法上可以像普通函数一样用()调用，同时还拥有对象携带状态、有明确类型的能力。

你写出`obj(a, b)`时：

- 如果obj是普通函数，就是直接调用函数；
- 如果obj是某个类的对象，编译器会自动转换为`obj.operator()(a, b)`，这就是重载()运算符的效果。

> Lambda表达式（C++11）：编译器会为每个Lambda自动生成一个匿名的函数对象。可以在代码中"就地"定义一个可调用对象，无需单独写一个类或函数。

## 1. 函数对象基础

函数对象本质上是重载了`operator()`的类/结构体。它的对象可以像普通函数一样被调用：`对象(参数)`。

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

// ===== 1. 无状态函数对象：打印器 =====
struct Printer
{
    // 重载()运算符，对象可像函数一样调用
    void operator()(int x) const
    {
        cout << x << " ";
    }
};

// ===== 2. 带状态函数对象：计数器 =====
struct Counter
{
    int count;
    Counter() : count(0) {}
    void operator()(int x)
    {
        if (x > 50) count++; // 统计大于50的元素数量
    }
};

// ===== 3. 参数化函数对象：阈值比较器 =====
struct GreaterThan
{
    int threshold;
    GreaterThan(int t) : threshold(t) {} // 构造函数传入阈值，定制判断逻辑
    bool operator()(int x) const
    {
        return x > threshold; // 判断元素是否大于阈值
    }
};

// ===== 4. 作为sort比较器的函数对象 =====
// 对pair，按first字符串降序比较
struct CompareByFirstDesc
{
    bool operator()(const pair<string, int>& a,
                    const pair<string, int>& b) const
    {
        return a.first > b.first;
    }
};

int main()
{
    vector<int> data;
    for (int i = 1; i <= 10; i++)
        data.push_back(i * 10);

    // 1. Printer：for_each传入临时函数对象 Printer()
    cout << "=== 1. Printer函数对象 ===" << endl;
    cout << "数据: ";
    for_each(data.begin(), data.end(), Printer());
    cout << endl;

    // 2. Counter：带内部状态；for_each按值传递，返回拷贝，需接收结果才能拿到count
    cout << "\n=== 2. Counter统计大于50的数 ===" << endl;
    Counter c;
    c = for_each(data.begin(), data.end(), c);
    cout << "大于50的数有 " << c.count << " 个" << endl;

    // 3. GreaterThan：构造时传入不同阈值，复用同一套判断逻辑
    cout << "\n=== 3. GreaterThan参数化比较 ===" << endl;
    int n = count_if(data.begin(), data.end(), GreaterThan(30));
    cout << "大于30的: " << n << " 个" << endl;
    n = count_if(data.begin(), data.end(), GreaterThan(70));
    cout << "大于70的: " << n << " 个" << endl;

    // 4. sort自定义比较器，用于pair排序
    cout << "\n=== 4. 自定义比较器排序 ===" << endl;
    vector<pair<string, int>> items;
    items.push_back(make_pair("加热炉", 85));
    items.push_back(make_pair("反应釜", 120));
    items.push_back(make_pair("冷却塔", 28));
    items.push_back(make_pair("阀门", 45));

    // 4a. pair默认排序：优先按first升序
    vector<pair<string, int>> sorted1 = items;
    sort(sorted1.begin(), sorted1.end());
    cout << "默认排序（按名称升序）:" << endl;
    for (size_t i = 0; i < sorted1.size(); i++)
        cout << "  " << sorted1[i].first << ": " << sorted1[i].second << endl;

    // 4b. 使用自定义函数对象，按名称降序
    vector<pair<string, int>> sorted2 = items;
    sort(sorted2.begin(), sorted2.end(), CompareByFirstDesc());
    cout << "CompareByFirstDesc（按名称降序）:" << endl;
    for (size_t i = 0; i < sorted2.size(); i++)
        cout << "  " << sorted2[i].first << ": " << sorted2[i].second << endl;

    return 0;
}
```

输出结果：

```
=== 1. Printer函数对象 ===
数据: 10 20 30 40 50 60 70 80 90 100

=== 2. Counter统计大于50的数 ===
大于50的数有 5 个

=== 3. GreaterThan参数化比较 ===
大于30的: 7 个
大于70的: 3 个

=== 4. 自定义比较器排序 ===
默认排序（按名称升序）:
  阀门: 45
  反应釜: 120
  加热炉: 85
  冷却塔: 28
CompareByFirstDesc（按名称降序）:
  冷却塔: 28
  加热炉: 85
  反应釜: 120
  阀门: 45
```

> 1. `operator()` 到底是什么

`operator()` 叫**函数调用运算符重载**。C++ 规定：`()` 是运算符，就像 `+`、`>` 一样，我们可以给类 / 结构体重载这个运算符。

```c++
struct Printer{
    void operator()(int x) const
    {
        cout << x << " ";
    }
};
```

当你写`Printer p; p(10);`，底层会被翻译成`p.operator()(10);`

一句话总结：`operator()` 就是让对象能使用 `()` 括号调用的重载函数

> 2. `Counter c; c = for_each(..., c);` 和 `for_each(..., Printer());` 两者的差别

- `Printer()`：**临时对象**，传给 for_each，用完直接丢弃，**不需要保存状态**。
- `Counter c`：**命名对象**，我们要利用它**保存不断变化的 count 状态**，而 for_each 是值拷贝，所以必须接收返回值拿回修改后的副本。

---

## 2. 标准库中的函数对象

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional> // plus/less/greater / std::bind

using namespace std;

int main(){
    vector<int> v;
    for (int i = 1; i <= 8; i++) v.push_back(i * 3 % 10 + 1);
    cout << "原始: ";
    for_each(v.begin(), v.end(), [](int x){ cout << x << " "; });
    cout << endl;

    // 1. less<T>：小于比较器，sort默认规则，a < b，升序排序
    sort(v.begin(), v.end(), less<int>());  // 与sort默认行为一致（升序）
    cout << "less<int>() 升序: ";
    for (size_t i = 0; i < v.size(); i++) cout << v[i] << " ";
    cout << endl;

    // 2. greater<T>：大于比较器，a > b，降序排序
    sort(v.begin(), v.end(), greater<int>());  // 降序
    cout << "greater<int>() 降序: ";
    for (size_t i = 0; i < v.size(); i++) cout << v[i] << " ";
    cout << endl;

    // 3. plus<T> 算术函数对象，配合 std::bind 绑定第二个参数
    // bind(plus<int>(), placeholders::_1, 10) => 传入的第一个参数 +10
    vector<int> result(v.size());
    transform(v.begin(), v.end(), result.begin(),
              bind(plus<int>(), placeholders::_1, 10));  // 每个元素 +10
    cout << "每个元素+10: ";
    for (size_t i = 0; i < result.size(); i++) cout << result[i] << " ";
    cout << endl;

    // 4. logical_and<T>/logical_or<T>/logical_not<T>：逻辑运算函数对象
    // 5. equal_to<T>/not_equal_to<T>：相等、不等比较函数对象
    // count_if + std::bind，绑定equal_to的第二个参数为5
    int count5 = count_if(v.begin(), v.end(),
                          bind(equal_to<int>(), placeholders::_1, 5));
    cout << "等于5的元素个数: " << count5 << endl;

    return 0;
}
```

输出结果：

```
原始: 4 7 10 3 6 9 2 5
less<int>() 升序: 2 3 4 5 6 7 9 10
greater<int>() 降序: 10 9 7 6 5 4 3 2
每个元素+10: 20 19 17 16 15 14 13 12
等于5的元素个数: 1
```

---

```c++
vector<int> result(v.size());
transform(v.begin(), v.end(), result.begin(),
          bind(plus<int>(), placeholders::_1, 10));
```

1. transform函数原型

```
transform(输入起始, 输入结束, 输出起始, 一元函数对象)
```

功能：遍历输入区间 `[输入起始,输入结束)`，**对每一个元素调用传入的函数**，把函数返回值写入输出容器。

一元函数对象是指一次只接受一个参数。

2. plus<int>是指

`plus<int>` 是 `<functional>` 提供的**标准二元函数对象**，它重载了 `operator()`，接收**两个参数**，返回相加结果。`plus<int>()(a, b)`  等价 return a + b;

问题：transform的第四个参数只能传入一个参数，因此需要bind把其中一个参数固定死。

3. bind(plus<int>(), placeholders::_1, 10)

`std::bind`：绑定器，包装一个可调用对象，固定部分参数，生成新的可调用对象。`bind( 可调用对象, 参数1, 参数2, ... )`。

- `plus<int>()`：要包装的二元函数对象，它需要两个参数 `(arg1, arg2)`。
- `placeholders::_1`：**占位符**。含义：将来调用这个 bind 包装后的函数时，**外面传进来的第 1 个参数放在这个位置**。
- `10`：固定常量，永远是第二个参数，不会变。

---

## 3. Lambda表达式（C++11）：匿名函数

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

int main(){
    vector<int> data;
    for (int i = 1; i <= 10; i++) data.push_back(i * 15 % 100);
    cout << "原始数据: ";
    for (size_t i = 0; i < data.size(); i++) cout << data[i] << " ";
    cout << endl;

    // ===== 1. 最简单Lambda：无捕获 =====
    cout << "\n=== 1. 简单Lambda打印 ===" << endl;
    for_each(data.begin(), data.end(),
             [](int x) { cout << x << " "; });  // Lambda表达式
    // Lambda完整语法：[捕获列表](参数列表) mutable 异常 -> 返回值类型 {函数体}
    // 常用简化写法：[捕获](参数) { 代码 }
    cout << endl;

    // ===== 2. Lambda配合sort作为比较器 =====
    cout << "\n=== 2. Lambda排序 ===" << endl;
    // 升序
    sort(data.begin(), data.end(),
         [](int a, int b) { return a < b; });
    cout << "升序: ";
    for (size_t i = 0; i < data.size(); i++) cout << data[i] << " ";
    cout << endl;
    // 降序
    sort(data.begin(), data.end(),
         [](int a, int b) { return a > b; });
    cout << "降序: ";
    for (size_t i = 0; i < data.size(); i++) cout << data[i] << " ";
    cout << endl;

    // ===== 3. 值捕获：拷贝外部变量，Lambda内部是副本，外部修改不影响 =====
    cout << "\n=== 3. 值捕获外部变量 ===" << endl;
    int threshold = 50;
    // [threshold] 值捕获：复制一份threshold到lambda
    int aboveThreshold = count_if(data.begin(), data.end(),
                                  [threshold](int x) { return x > threshold; });
    cout << "大于 " << threshold << " 的有 " << aboveThreshold << " 个" << endl;

    threshold = 30;
    int above30 = count_if(data.begin(), data.end(),
                           [threshold](int x) { return x > threshold; });
    cout << "大于 " << threshold << " 的有 " << above30 << " 个" << endl;

    // ===== 4. 引用捕获：&捕获，直接操作外部原变量，可以修改外部值 =====
    cout << "\n=== 4. 引用捕获并修改外部变量 ===" << endl;
    int sum = 0;
    for_each(data.begin(), data.end(),
             [&sum](int x) { sum += x; }); // &sum 引用捕获
    cout << "累加和: " << sum << endl;

    // ===== 5. 显式指定返回值类型 -> 类型 =====
    cout << "\n=== 5. 显式返回值类型 ===" << endl;
    double avg = 0;
    if (!data.empty())
    {
        int total = 0;
        for_each(data.begin(), data.end(), [&total](int x) { total += x; });
      	// static_cast 是 C++ 的静态类型转换关键字
      	// avg = (double)total / data.size(); 这是C风格的代码
        avg = static_cast<double>(total) / data.size();  // C++风格代码
    }
    cout << "平均值: " << avg << endl;

    // ===== 6. 混合捕获：部分引用、部分值捕获 =====
    cout << "\n=== 6. 混合捕获 ===" << endl;
    int minLimit = 20, maxLimit = 80;
    int outOfRangeCount = 0;
    // [&outOfRangeCount, minLimit, maxLimit]
    // outOfRangeCount引用捕获；minLimit、maxLimit值捕获
    for_each(data.begin(), data.end(),
             [&outOfRangeCount, minLimit, maxLimit](int x) {
                 if (x < minLimit || x > maxLimit) outOfRangeCount++;
             });
    cout << "超出[" << minLimit << ", " << maxLimit << "]的数: "
         << outOfRangeCount << " 个" << endl;

    return 0;
}
```

输出结果：

```
原始数据: 15 30 45 60 75 90 5 20 35 50

=== 1. 简单Lambda打印 ===
15 30 45 60 75 90 5 20 35 50

=== 2. Lambda排序 ===
升序: 5 15 20 30 35 45 50 60 75 90
降序: 90 75 60 50 45 35 30 20 15 5

=== 3. 值捕获外部变量 ===
大于 50 的有 3 个
大于 30 的有 6 个

=== 4. 引用捕获并修改外部变量 ===
累加和: 425

=== 5. 显式返回值类型 ===
平均值: 42.5

=== 6. 混合捕获 ===
超出[20, 80]的数: 3 个
```

> Lambda 核心知识点

**Lambda 本质：编译器自动生成的匿名函数对象（仿函数）**，自动重载`operator()`，省去手动写 struct 的麻烦。 语法：`[捕获列表](参数列表) -> 返回类型 {函数体}`

> 捕获列表规则（重点）

1. `[]`：不捕获任何外部变量；
2. `[var]`：**值捕获**，拷贝变量到 lambda 内部；lambda 拿到的是副本，外部后续修改不会影响 lambda 内的值，默认不能修改副本（加`mutable`才可以改副本）；
3. `[&var]`：**引用捕获**，直接绑定外部原变量，lambda 内部修改就是修改外部变量；
4. `[=]`：**全部值捕获**，所有外部变量都拷贝；
5. `[&]`：**全部引用捕获**，所有外部变量都引用；
6. 混合捕获：`[&a, b]`：a 引用捕获，b 值捕获；
7. `[this]`：捕获this指针，可访问类成员；

> 适用场景

直接作为 STL 算法（`sort` / `for_each` / `count_if`）的可调用参数，替代手写仿函数、函数指针，代码更简短。

---

## 4. Lambda与函数对象：编译器的魔法

```c++
// 原始Lambda代码
int threshold = 50;
count_if(data.begin(), data.end(),
         [threshold](int x) { return x > threshold; });

// 编译器大致翻译成下面这个匿名仿函数（简化示意）
struct AnonymousFunctor
{
    int threshold;        // 值捕获的变量，变成仿函数的成员变量
    AnonymousFunctor(int t) : threshold(t) {} // 构造函数，拷贝外部变量

    // operator() 默认带 const，因为值捕获默认不允许修改内部成员
    bool operator()(int x) const
    {
        return x > threshold;
    }
};
count_if(data.begin(), data.end(), AnonymousFunctor(threshold));
```

> 知识点理解：

1. **`[threshold]` 值捕获**：编译器生成仿函数时，**把外部变量拷贝一份，存为仿函数的成员变量**。调用仿函数构造器时，把外部`threshold`的值复制进去。
2. `operator()` 后面自带 `const`：默认情况下，lambda**不能修改值捕获得到的成员变量**。
   - 如果想在 lambda 内部修改捕获的副本，需要加 `mutable`：`[threshold](int x) mutable { threshold++; return x > threshold; }`
   - 加上`mutable`后，生成的`operator()`就会去掉`const`。
3. 外部后续修改`threshold`，**不会影响仿函数内部的成员**，因为是拷贝，不是引用。
4. `AnonymousFunctor(threshold)` **不是赋值，是创建临时对象（调用构造函数）**。 `AnonymousFunctor` 是结构体名，`AnonymousFunctor(threshold)` 等价：**调用结构体的构造函数，生成一个匿名临时仿函数对象**。

## 5. mutable Lambda：修改捕获的值

默认情况下，Lambda的`operator()`是const的，不能修改值捕获的变量。加`mutable`可以。

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main(){
    vector<int> v;
    for (int i = 1; i <= 5; i++) v.push_back(i);

    // ===== 值捕获：不加mutable，无法修改捕获的副本 =====
    int x = 0;
    // 下面代码编译报错：lambda默认生成const的operator()，不能修改成员
    // for_each(v.begin(), v.end(), [x](int n) { x += n; });

    // 值捕获 + mutable：允许lambda内部修改捕获到的副本
    // 注意：修改的只是lambda内部拷贝出来的副本，外部原始x不受影响
    for_each(v.begin(), v.end(),
             [x](int n) mutable {
                 x += n;        // mutable放开const限制，可以修改副本x
                 cout << "内部x=" << x << " ";
             });
    cout << "\n外部x=" << x << " (值捕获，修改的是副本，外部不变)" << endl;

    // ===== 引用捕获：直接操作外部原变量，不需要mutable =====
    int y = 0;
    for_each(v.begin(), v.end(),
             [&y](int n) { y += n; }); // 引用捕获，操作的就是外部原始y
    cout << "引用捕获后y=" << y << endl;

    return 0;
}
```

输出结果：

```
内部x=1 内部x=3 内部x=6 内部x=10 内部x=15
外部x=0 (值捕获，修改的是副本，外部不变)
引用捕获后y=15
```

---

## 6. Lambda作为返回值/参数（函数式编程风格）

> `std::function<T>`是C++11引入的"可调用对象包装器"，它可以容纳任何能像函数一样调用的东西：普通函数、函数指针、函数对象、Lambda。这让我们可以用统一的接口来处理不同的可调用对象。

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>   // std::function 可调用对象包装器
#include <string>
using namespace std;

// ===== 1. std::function<T>：统一包装各类可调用对象 =====
// std::function<bool(int)> 可以包装任意：接收1个int、返回bool的可调用对象
// 可调用对象：普通函数、函数对象、Lambda表达式

// ===== 2. Lambda作为函数返回值 =====
function<bool(int)> makeThresholdChecker(int threshold)
{
    // 返回Lambda，值捕获threshold
    return [threshold](int x) { return x > threshold; };
}

// ===== 3. std::function作为函数参数，统一接口 =====
void processData(const vector<int>& data,
                 function<bool(int)> condition,   // 判断条件
                 function<void(int)> handler)     // 满足条件后的处理动作
{
    for (size_t i = 0; i < data.size(); i++)
    {
        if (condition(data[i]))
            handler(data[i]);
    }
}

int main(){
    vector<int> readings;
    for (int i = 0; i < 10; i++) readings.push_back(15 + i * 8);
    cout << "读数: ";
    for (size_t i = 0; i < readings.size(); i++) cout << readings[i] << " ";
    cout << endl;

    // 1. 调用工厂函数，返回Lambda并保存到function对象
  	// auto是指什么呢
    cout << "\n=== 使用工厂函数创建检查器 ===" << endl;
    auto above50 = makeThresholdChecker(50);
    auto above80 = makeThresholdChecker(80);
    int count50 = 0, count80 = 0;
    for (size_t i = 0; i < readings.size(); i++)
    {
        if (above50(readings[i])) count50++;
        if (above80(readings[i])) count80++;
    }
    cout << ">50 的有 " << count50 << " 个, >80 的有 " << count80 << " 个" << endl;
		
    // 2. processData接收两个Lambda，作为条件和处理函数
    cout << "\n=== 使用processData处理数据 ===" << endl;
    cout << "报警值(>80): ";
    processData(readings,
                [](int x) { return x > 80; },         // condition判断
                [](int x) { cout << x << "! "; });    // handler处理
    cout << endl;
    cout << "正常范围[30,80]: ";
    processData(readings,
                [](int x) { return x >= 30 && x <= 80; },
                [](int x) { cout << x << " "; });
    cout << endl;

    // 3. Lambda作为sort比较器，随时更换排序规则
    vector<pair<string, int>> devices;
    devices.push_back(make_pair("加热炉A", 85));
    devices.push_back(make_pair("反应釜C", 120));
    devices.push_back(make_pair("冷却塔B", 28));
    devices.push_back(make_pair("阀门D", 45));
    cout << "\n=== Lambda灵活比较 ===" << endl;
    // 按数值升序
  	// sort：返回 true/false，告诉 sort：原始容器里这两个元素谁该放前面；sort 根据返回结果，对容器里面原始元素进行交换重排。如果用const pair<string, int> a，会复制一份元素去做比较，然后由sort去调整devices的中的排序
    sort(devices.begin(), devices.end(),
         [](const pair<string, int>& a, const pair<string, int>& b) {
             return a.second < b.second;
         });
    cout << "按数值升序: ";
    for (size_t i = 0; i < devices.size(); i++) cout << devices[i].first << " ";
    cout << endl;
    // 按名称字符串长度升序
    sort(devices.begin(), devices.end(),
         [](const pair<string, int>& a, const pair<string, int>& b) {
             return a.first.size() < b.first.size();
         });
    cout << "按名称长度: ";
    for (size_t i = 0; i < devices.size(); i++) cout << devices[i].first << " ";
    cout << endl;

    return 0;
}
```

输出结果：

```
读数: 15 23 31 39 47 55 63 71 79 87

=== 使用工厂函数创建检查器 ===
>50 的有 5 个, >80 的有 1 个

=== 使用processData处理数据 ===
报警值(>80): 87!
正常范围[30,80]: 31 39 47 55 63 71 79

=== Lambda灵活比较 ===
按数值升序: 冷却塔B 阀门D 加热炉A 反应釜C
按名称长度: 阀门D 冷却塔B 加热炉A 反应釜C
```


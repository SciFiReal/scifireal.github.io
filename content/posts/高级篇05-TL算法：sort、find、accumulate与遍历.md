---
title: "高级篇05-STL算法：sort、find、accumulate与遍历"

date: 2026-10-08T17:53:33+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇05-STL算法：sort、find、accumulate与遍历"

toc: true
---



> `STL`的第二大组件——算法，为何会有算法呢，之前学习手动写循环处理容器中数据容易出错。而算法提供了几十种通用算法来操作容器中的数据：排序，查找，遍历，转换，统计等。一次实现，任何容器都能用。

> 算法是通过迭代器来实现的，迭代器类似指针，去访问容器上的数据。

> **主要算法分类**：

1. **排序**：sort、stable_sort、partial_sort
2. **查找**：find、find_if、binary_search、lower_bound
3. **遍历/转换**：for_each、transform
4. **统计**：count、count_if、accumulate
5. **拷贝/移动**：copy、copy_if、move
6. **删除/修改**：remove、remove_if、replace、fill
7. **数值算法**：accumulate、inner_product、partial_sum

---

## 1. 迭代器的基本用法

```c++
#include <iostream>
#include <vector>
#include <list>
#include <string>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30, 40, 50};

    // 1. 迭代器基础操作：指向、偏移、解引用
    vector<int>::iterator it;
    it = v.begin();        // 指向容器首个元素
    cout << "第一个元素: " << *it << endl;

    ++it;                  // 迭代器后移，指向第二个元素
    cout << "第二个元素: " << *it << endl;

    it = v.end();          // 指向容器末尾的后一位（不存储元素）
    --it;                  // 迭代器前移，指向最后一个元素
    cout << "最后一个元素: " << *it << endl;

    // 2. 普通迭代器遍历容器（可读写元素）
    cout << "\n迭代器遍历: ";
    for (vector<int>::iterator i = v.begin(); i != v.end(); ++i) {
        cout << *i << " ";
    }
    cout << endl;

    // 3. const只读迭代器（仅访问，不可修改元素）
    cout << "const迭代器遍历: ";
    for (vector<int>::const_iterator i = v.begin(); i != v.end(); ++i) {
        // *i = 100;  // 报错：只读迭代器禁止修改元素
        cout << *i << " ";
    }
    cout << endl;

    // 4. 反向迭代器：从后往前遍历容器
    cout << "反向迭代器遍历: ";
    for (vector<int>::reverse_iterator i = v.rbegin(); i != v.rend(); ++i) {
        cout << *i << " ";
    }
    cout << endl;

    // 5. list容器迭代器（STL容器迭代器接口统一，用法一致）
    list<string> names = {"加热炉A", "冷却塔B", "反应釜C"};
    cout << "list容器迭代器遍历: ";
    for (list<string>::iterator i = names.begin(); i != names.end(); ++i) {
        cout << *i << " ";
    }
    cout << endl;

    // 6. 数组天然迭代器：指针等价于数组迭代器
    int arr[] = {1, 2, 3, 4, 5};
    cout << "数组指针迭代遍历: ";
    for (int* p = arr; p != arr + 5; ++p) {
        cout << *p << " ";
    }
    cout << endl;

    return 0;
}
```

输出结果：

```
第一个元素: 10
第二个元素: 20
最后一个元素: 50

迭代器遍历: 10 20 30 40 50
const迭代器遍历: 10 20 30 40 50
反向迭代器遍历: 50 40 30 20 10
list容器迭代器遍历: 加热炉A 冷却塔B 反应釜C
数组指针迭代遍历: 1 2 3 4 5
```

> 核心知识点总结

- 基础迭代器：`begin()`指向首元素，`end()`指向末尾后位置，通过`++/--`偏移。
- const迭代器：只读属性，不能修改容器中的数据。
- 反向迭代器：`rbegin()`（末尾）、`rend()`（首位前），实现倒序遍历。
- `STL`统一接口：`vector、list`等所有`STL`容器迭代器用法通用。
- 数组迭代特性：普通数组指针具备迭代能力。

---

## 2. 排序算法：sort

```c++
#include <iostream>
#include <vector>
#include <algorithm>  // sort算法必备头文件
#include <cstdlib>
#include <ctime>
#include <string>
using namespace std;

// 自定义比较函数：设备温度降序排序
// pair<string, double>：pair是STL中二元组的写法，把两个不同数据类型打包在一起，形成一个整体
bool compareByTempDesc(const pair<string, double>& a, 
                       const pair<string, double>& b)
{
  	// second表示第二个元素，first是第一个元素
    return a.second > b.second; 
}

int main()
{
    srand(time(0));  // 设置随机数种子

    // 1. 基础用法：默认升序排序
    vector<int> nums;
    // 生成10个0-99随机数
    for (int i = 0; i < 10; i++) 
        nums.push_back(rand() % 100);

    cout << "排序前: ";
    for (size_t i = 0; i < nums.size(); i++) 
        cout << nums[i] << " ";
    
    sort(nums.begin(), nums.end()); // 默认从小到大升序
    cout << "\n升序排序后: ";
    for (size_t i = 0; i < nums.size(); i++) 
        cout << nums[i] << " ";

    // 2. 库函数降序排序：使用greater<数据类型>()：用于比较int类型，判断a>b是降序
    sort(nums.begin(), nums.end(), greater<int>());
    cout << "\n降序排序后: ";
    for (size_t i = 0; i < nums.size(); i++) 
        cout << nums[i] << " ";

    // 3. 自定义规则排序：pair设备温度排序
    vector<pair<string, double>> devices = {
        {"加热炉A", 78.5},
        {"冷却塔B", 28.3},
        {"反应釜C", 125.8},
        {"阀门D", 45.2}
    };
    // 根据自定义函数按温度降序排序
    sort(devices.begin(), devices.end(), compareByTempDesc);

    cout << "\n\n按温度降序排列的设备:" << endl;
    for (size_t i = 0; i < devices.size(); i++)
    {
        cout << "  " << i + 1 << ". " << devices[i].first
             << " (" << devices[i].second << "度)" << endl;
    }

    // 4. 字符串排序：默认按字典序升序
    vector<string> strs = {"device", "sensor", "alarm", "temperature", "system"};
    sort(strs.begin(), strs.end());
    cout << "\n字符串字典序排序: ";
    for (size_t i = 0; i < strs.size(); i++) 
        cout << strs[i] << " ";

    return 0;
}
```

输出结果：

```
排序前: 88 19 25 33 61 17 95 23 90 58
升序排序后: 17 19 23 25 33 58 61 88 90 95
降序排序后: 95 90 88 61 58 33 25 23 19 17

按温度降序排列的设备:
  1. 反应釜C (125.8度)
  2. 加热炉A (78.5度)
  3. 阀门D (45.2度)
  4. 冷却塔B (28.3度)

字符串字典序排序: alarm device sensor system temperature
```

> sort的特点：

- 平均O(nlogn)时间复杂度，是快排的变体；
- vector/deque/数组支持，list不支持，需用list自带的sort方法；
- 不稳定排序，相等元素的相对顺序可能改变，需要稳定排序用`stabel_sort`；

---

## 3. 查找算法：find / find_if / binary_search

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

// 自定义设备结构体
struct Device{
    int id;
    string name;
    double temperature;
    // 构造函数，快速初始化设备
    Device(int i, string n, double t) : id(i), name(n), temperature(t) {}
};

// 查找判断条件：温度大于80度
bool isOverTemp(const Device& d){
    return d.temperature > 80.0;
}

int main(){
    vector<Device> devices;
    devices.push_back(Device(1001, "加热炉A", 78.5));
    devices.push_back(Device(1002, "冷却塔B", 28.3));
    devices.push_back(Device(2001, "反应釜C", 125.8));
    devices.push_back(Device(1003, "加热炉B", 85.0));
    devices.push_back(Device(3001, "阀门D", 45.2));

    // 1. find：查找【等于目标值】的元素
    vector<int> ids = {1001, 1002, 2001, 1003, 3001};
    vector<int>::iterator found = find(ids.begin(), ids.end(), 2001);
    if (found != ids.end())
        cout << "找到 2001，位置索引: " << (found - ids.begin()) << endl;
    else
        cout << "未找到 2001" << endl;

    // 2. find_if：查找**第一个满足自定义条件**的元素，迭代器it是指针
    cout << "\n查找第一个超过80度的设备:" << endl;
    vector<Device>::iterator it = find_if(devices.begin(), devices.end(), isOverTemp);
    if (it != devices.end())
        cout << "找到: " << it->name << " (" << it->temperature << "度)" << endl;

    // 3. find_if循环：查找**全部满足条件**的元素
    cout << "\n所有超过80度的设备:" << endl;
    vector<Device>::iterator current = devices.begin();
    while ((current = find_if(current, devices.end(), isOverTemp)) != devices.end())
    {
        cout << "  " << current->name << ": " << current->temperature << "度" << endl;
        ++current; // 移动到下一个元素，继续往后搜索
    }

    // 4. binary_search 二分查找：快速判断元素是否存在【前提：容器必须预先排序】
    sort(ids.begin(), ids.end());
    cout << "\n二分查找:" << endl;
    if (binary_search(ids.begin(), ids.end(), 1003))
        cout << "  1003 存在" << endl;
    else
        cout << "  1003 不存在" << endl;

    // 5. count / count_if：统计元素数量
    cout << "\n统计:" << endl;
    int totalOver = count_if(devices.begin(), devices.end(), isOverTemp);
    cout << "超过80度的设备: " << totalOver << "个" << endl;

    return 0;
}
```

输出结果：

```
找到 2001，位置索引: 2

查找第一个超过80度的设备:
找到: 反应釜C (125.8度)

所有超过80度的设备:
  反应釜C: 125.8度
  加热炉B: 85度

二分查找:
  1003 存在

统计:
超过80度的设备: 2个
```

> 都在<algorithm>头文件，全部接受迭代器区间[begin, end)

---

## 4. 遍历与转换：for_each / transform

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
#include <cctype>
using namespace std;

// for_each 的回调函数：打印设备信息
void printDevice(const pair<string, double>& d)
{
    cout << "  " << d.first << ": " << d.second << "度" << endl;
}

// transform回调：给设备名追加报警标签
string addAlertTag(const pair<string, double>& d)
{
    if (d.second > 80.0)
        return d.first + " (ALARM)";
    return d.first + " (正常)";
}

// transform回调：温度转为状态码
int tempToStatus(double t)
{
    if (t > 80) return 2;  // 报警
    if (t > 70) return 1;  // 警告
    return 0;              // 正常
}

int main()
{
    vector<pair<string, double>> devices;
    devices.push_back(make_pair("加热炉A", 78.5));
    devices.push_back(make_pair("冷却塔B", 28.3));
    devices.push_back(make_pair("反应釜C", 125.8));
    devices.push_back(make_pair("阀门D", 45.2));

    // 1. for_each：遍历区间，每个元素执行同一个函数
    cout << "=== for_each遍历 ===" << endl;
    for_each(devices.begin(), devices.end(), printDevice);

    // 2. transform：转换容器元素，输出到新容器（back_inserter尾部追加）
    cout << "\n=== transform添加报警标记 ===" << endl;
    vector<string> taggedNames;
    transform(devices.begin(), devices.end(),
              back_inserter(taggedNames),
              addAlertTag);
    for (size_t i = 0; i < taggedNames.size(); i++)
        cout << "  " << taggedNames[i] << endl;

    // 3. transform：提取温度，映射成状态码
    cout << "\n=== transform：温度→状态码 ===" << endl;
    vector<double> temps;
    for (size_t i = 0; i < devices.size(); i++)
        temps.push_back(devices[i].second);
    vector<int> statuses;
    transform(temps.begin(), temps.end(),
              back_inserter(statuses),
              tempToStatus);
    cout << "温度: ";
    for (size_t i = 0; i < temps.size(); i++) cout << temps[i] << " ";
    cout << "\n状态: ";
    for (size_t i = 0; i < statuses.size(); i++) cout << statuses[i] << " ";
    cout << "\n(0=正常, 1=警告, 2=报警)" << endl;

    // 4. transform：字符串大小写转换示例
    cout << "\n=== transform：字符串大写 ===" << endl;
    string name = "temperature_sensor";
    string upperName;
    for (size_t i = 0; i < name.size(); i++)
        upperName.push_back(toupper(name[i]));
    cout << name << " → " << upperName << endl;

    return 0;
}
```

输出结果：

```
=== for_each遍历 ===
  加热炉A: 78.5度
  冷却塔B: 28.3度
  反应釜C: 125.8度
  阀门D: 45.2度

=== transform添加报警标记 ===
  加热炉A (正常)
  冷却塔B (正常)
  反应釜C (ALARM)
  阀门D (正常)

=== transform：温度→状态码 ===
温度: 78.5 28.3 125.8 45.2
状态: 1 0 2 0
(0=正常, 1=警告, 2=报警)

=== transform：字符串大写 ===
temperature_sensor → TEMPERATURE_SENSOR
```

---

## 5. 数值算法：accumulate

```c++
#include <iostream>
#include <vector>
#include <numeric>  // accumulate 头文件
#include <string>
using namespace std;

int main()
{
    // 1. 基础数值累加，第三个参数为累加初值
    vector<int> readings;
    for (int i = 1; i <= 10; i++) readings.push_back(i * 10);
    int total = accumulate(readings.begin(), readings.end(), 0);
    double avg = (double)total / readings.size();

    cout << "读数序列: ";
    for (size_t i = 0; i < readings.size(); i++) cout << readings[i] << " ";
    cout << "\n累加和: " << total << ", 平均值: " << avg << endl;

    // 2. 浮点累加：初值写成0.0（double类型，决定返回值类型）
    vector<double> temps = {78.5, 85.3, 125.8, 28.3};
    double totalTemp = accumulate(temps.begin(), temps.end(), 0.0);
    cout << "\n温度合计: " << totalTemp
         << "度, 平均: " << totalTemp / temps.size() << "度" << endl;

    // 3. 第四个参数：自定义运算，示例求最大值
    double maxT = accumulate(temps.begin(), temps.end(),
                            temps[0],
                            [](double a, double b) { return a > b ? a : b; });
    cout << "最高温度: " << maxT << "度" << endl;

    // 4. accumulate通用能力：字符串拼接
    vector<string> tags = {"温度正常", "压力正常", "流量异常", "阀门正常"};
    string statusReport = accumulate(
        tags.begin(), tags.end(),
        string("设备状态: "),
        [](const string& acc, const string& s) {
            return acc + s + "; ";
        });
    cout << "\n状态报告: " << statusReport << endl;

    return 0;
}
```

输出结果：

```
读数序列: 10 20 30 40 50 60 70 80 90 100
累加和: 550, 平均值: 55

温度合计: 317.9度, 平均: 79.475度
最高温度: 125.8度

状态报告: 设备状态: 温度正常; 压力正常; 流量异常; 阀门正常;
```

---

## 6. 其他常用算法：copy / remove / replace / fill

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>
using namespace std;

// 条件判断函数：数值大于80触发告警替换
bool isAlertValue(int v) { 
    return v > 80; 
}

int main()
{
    vector<int> data;
    // 生成10、20...100的测试数据
    for (int i = 1; i <= 10; i++) 
        data.push_back(i * 10);

    cout << "原始数据: ";
    for (int val : data) cout << val << " ";
    cout << endl;

    // 1. copy：完整复制容器元素到目标容器
    vector<int> copyTarget(data.size());
    copy(data.begin(), data.end(), copyTarget.begin());
    cout << "copy复制后: ";
    for (int val : copyTarget) cout << val << " ";
    cout << endl;

    // 2. replace：精准替换 指定值 → 新值
    replace(data.begin(), data.end(), 50, 0);
    cout << "replace(50→0): ";
    for (int val : data) cout << val << " ";
    cout << endl;

    // 3. replace_if：按自定义条件批量替换
    replace_if(data.begin(), data.end(), isAlertValue, 999);
    cout << "replace_if(>80→999): ";
    for (int val : data) cout << val << " ";
    cout << endl;

    // 4. fill：用固定值批量填充整个容器
    vector<int> buf(20);
    fill(buf.begin(), buf.end(), 42);
    cout << "fill全填充42: ";
    for (int val : buf) cout << val << " ";
    cout << endl;

    // 5. remove / remove_if + erase：真正删除容器元素
    vector<int> numbers;
    for (int i = 0; i < 10; i++) numbers.push_back(i);
    cout << "\n原始numbers: ";
    for (int val : numbers) cout << val << " ";

    // remove：移除指定值（仅前移有效元素，返回新末尾迭代器）
    vector<int>::iterator newEnd = remove(numbers.begin(), numbers.end(), 5);
    numbers.erase(newEnd, numbers.end()); // erase彻底删除冗余元素
    cout << "\nremove删除5后: ";
    for (int val : numbers) cout << val << " ";

    // 最简写法：remove_if + erase 链式删除（按条件删除偶数）
    numbers.erase(remove_if(numbers.begin(), numbers.end(),
        [](int x) { return x % 2 == 0; }),
        numbers.end());
    cout << "\n删除所有偶数后: ";
    for (int val : numbers) cout << val << " ";
    cout << endl;

    // 6. reverse：反转容器元素顺序
    reverse(data.begin(), data.end());
    cout << "\nreverse反转后: ";
    for (int val : data) cout << val << " ";
    cout << endl;

    return 0;
}
```

输出结果：

```
原始数据: 10 20 30 40 50 60 70 80 90 100
copy复制后: 10 20 30 40 50 60 70 80 90 100
replace(50→0): 10 20 30 40 0 60 70 80 90 100
replace_if(>80→999): 10 20 30 40 0 60 70 80 999 999
fill全填充42: 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42

原始numbers: 0 1 2 3 4 5 6 7 8 9
remove删除5后: 0 1 2 3 4 6 7 8 9
删除所有偶数后: 1 3 7 9

reverse反转后: 999 999 80 70 60 0 40 30 20 10
```

```c++
auto newEnd = remove(numbers.begin(), numbers.end(), 5);
```

1. `remove` 遍历，跳过 5，把其他元素向前覆盖移动 移动后数组内存变成：`[0,1,2,3,4,6,7,8,9,5]`

   > 注意：vector 的size 仍然是 10，最后那个`5`是残留垃圾，只是不再属于有效数据。

2. `newEnd` 这个迭代器，指向**第一个无效位置**（也就是原来 6,7,8,9 后面，垃圾元素的起点）

3. `numbers.erase(newEnd, numbers.end());`

   - `erase(迭代器起点, 迭代器终点)`：删除**区间 [newEnd , end)** 的所有元素
   - 执行后 vector 的 size 变成 9，丢掉后面的垃圾，最终 `[0,1,2,3,4,6,7,8,9]`

重点区分：

- `remove`：**逻辑删除**，移动元素，不改变 size，不改内存分配，只返回新边界
- `erase`：**物理删除**，释放后面元素，修改 vector 的 size

> 所有 STL 删除算法均遵循（先移动、再erase）原则，单独使用remove无效！

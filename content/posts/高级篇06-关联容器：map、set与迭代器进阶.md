---
title: "高级篇06-关联容器：map、set与迭代器进阶"

date: 2026-10-08T20:40:41+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇06-关联容器：map、set与迭代器进阶"

toc: true
---



> 上两篇学的vector/list/deque是**序列容器**（Sequence Containers）——元素按插入顺序排列
>
> STL还提供另一大家族：**关联容器**（Associative Containers）——元素按键（Key）自动排序，通过key快速查找。

> 主要的关联容器：

- **map**：键值对（key-value），key唯一且自动排序
- **set**：只有键（元素本身就是键），无重复

- **multimap**：map的变体，允许重复key
- **multiset**：set的变体，允许重复元素

> 关联容器的底层实现：通常是红黑树，查找/插入/删除都是O(log n)——比vector线性查找O(n)快得多，但比随机访问O(1)慢。
>
> 适用场景：
>
> - 需要按键查找（按用户名查找用户数据）
> - 需要自动排序（元素按key有序存储，遍历就是有序输出）
> - 不允许重复（set/map）或允许重复（multimap/multiset）

## 1. map：键值对（字典）

map存储`pair<const Key, Value>`，key唯一，自动按key从小到大排序，底层红黑树。

```c++
#include <iostream>
#include <map>    // map容器头文件
#include <string>
using namespace std;

// 设备信息结构体：存储一台设备的信息
struct DeviceInfo
{
    string name;        // 设备名称
    double temperature; // 温度
    bool isOnline;      // 是否在线

    // 构造函数，给成员默认值，方便快速创建对象
    DeviceInfo(string n = "", double t = 0, bool online = false)
        : name(n), temperature(t), isOnline(online) {}
};

int main()
{
    // ===================== 1. map插入数据 =====================
    // 定义map：key是int设备编号，value是DeviceInfo设备信息
    map<int, DeviceInfo> devices;

    // 方式1：[] 下标插入
    // ✅ 如果key不存在：新建这条数据
    // ✅ 如果key已经存在：**覆盖旧数据**
    devices[1001] = DeviceInfo("加热炉A", 78.5, true);
    devices[1002] = DeviceInfo("冷却塔B", 28.3, true);
    devices[2001] = DeviceInfo("反应釜C", 125.8, false);

    // 方式2：insert插入
    // ✅ 如果key不存在：插入成功
    // ✅ 如果key已经存在：**不会覆盖原有数据**（重点！和[]最大区别）
    // insert返回pair：第一个元素是迭代器，第二个bool标记是否插入成功
    pair<map<int, DeviceInfo>::iterator, bool> result;
    result = devices.insert(make_pair(3001, DeviceInfo("阀门D", 45.2, true)));
    if (result.second)
        cout << "3001插入成功" << endl;
    else
        cout << "3001已存在，未覆盖" << endl;

    // 重复insert 1001，1001已经存在，插入失败，旧数据保留
    result = devices.insert(make_pair(1001, DeviceInfo("XXX", 0, false)));
    cout << "再次insert 1001: " << (result.second ? "成功" : "失败(已存在)") << endl;


    // ===================== 2. 根据key查找数据 =====================
    cout << "\n=== 按键查找 ===" << endl;

    // 方式1：[]下标访问（坑！）
    // 如果key不存在，[]会**自动插入一条默认构造的空数据**，很多bug来源于此！
    cout << "1001: " << devices[1001].name << " T="
         << devices[1001].temperature << "度" << endl;

    // 方式2：at() 访问（C++11起，安全）
    // key不存在 → 直接抛出异常 out_of_range，不会偷偷新增元素
    try
    {
        cout << "2001: " << devices.at(2001).name << endl;
        cout << "9999: " << devices.at(9999).name << endl; // 不存在，触发异常
    }
    catch (const out_of_range& e)
    {
        cout << "at()异常: " << e.what() << endl;
    }

    // 方式3：find() 查找【工程最推荐写法】
    // 找到：返回指向该元素迭代器；找不到：返回 devices.end()
    map<int, DeviceInfo>::iterator it = devices.find(1002);
    if (it != devices.end())
        cout << "find(1002): " << it->second.name << endl;
    else
        cout << "find(1002): 未找到" << endl;

    it = devices.find(9999);
    cout << "find(9999): " << (it != devices.end() ? "找到" : "未找到") << endl;


    // ===================== 3. 遍历map =====================
    cout << "\n=== 遍历（map自动按key升序排列） ===" << endl;
    // map的元素是 pair<key类型, value类型>
    // i->first 就是key(设备ID)；i->second 就是value(DeviceInfo对象)
    for (map<int, DeviceInfo>::iterator i = devices.begin();
         i != devices.end();
         ++i)
    {
        cout << "  ID=" << i->first
             << " | " << i->second.name
             << " T=" << i->second.temperature << "度"
             << " (" << (i->second.isOnline ? "在线" : "离线") << ")" << endl;
    }
  	/* C++17适用auto简洁迭代器的写法
  	for (auto &i : devices)
		{
    		cout << "ID=" << i.first << " name=" << i.second.name << endl;
		}

  	*/


    // ===================== 4. 删除元素 =====================
    cout << "\n=== 删除 ===" << endl;
    // erase(键值)：按key删除，返回删除成功的个数（map的key唯一，返回0或1）
    size_t removed = devices.erase(1002);
    cout << "erase(1002): 删除了 " << removed << " 个元素" << endl;

    // erase(迭代器)：按迭代器删除，找到元素再删，更高效
    it = devices.find(3001);
    if (it != devices.end())
    {
        devices.erase(it);
        cout << "erase(迭代器): 删除3001" << endl;
    }

    cout << "删除后剩 " << devices.size() << " 台设备:" << endl;
    for (map<int, DeviceInfo>::iterator i = devices.begin();
         i != devices.end();
         ++i)
    {
        cout << "  " << i->first << " - " << i->second.name << endl;
    }


    // ===================== 5. 常用辅助接口 =====================
    cout << "\n设备总数: " << devices.size() << endl;
    cout << "是否为空: " << (devices.empty() ? "是" : "否") << endl;

    // count(key)：统计key存在数量。map中key唯一，返回只能是0或者1
    cout << "1001是否存在: " << (devices.count(1001) > 0 ? "是" : "否") << endl;
    cout << "9999是否存在: " << (devices.count(9999) > 0 ? "是" : "否") << endl;

    return 0;
}
```

输出结果：

```
3001插入成功
再次insert 1001: 失败(已存在)

=== 按键查找 ===
1001: 加热炉A T=78.5度
2001: 反应釜C
at()异常: invalid map<K, T> key
find(1002): 冷却塔B
find(9999): 未找到

=== 遍历（map自动按key升序排列） ===
  ID=1001 | 加热炉A T=78.5度 (在线)
  ID=1002 | 冷却塔B T=28.3度 (在线)
  ID=2001 | 反应釜C T=125.8度 (离线)
  ID=3001 | 阀门D T=45.2度 (在线)

=== 删除 ===
erase(1002): 删除了 1 个元素
erase(迭代器): 删除3001
删除后剩 2 台设备:
  1001 - 加热炉A
  2001 - 反应釜C

设备总数: 2
是否为空: 否
1001是否存在: 是
9999是否存在: 否
```

---

## 2. set：集合（只有键）

set存储单一值，元素自动排序且不重复，底层红黑树，复杂度O(log n)。

```c++
#include <iostream>
#include <set>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    // ===== 1. set 基本插入 =====
    // set：元素唯一，自动升序排序，底层红黑树
    set<int> ids;
    ids.insert(1001);
    ids.insert(1002);
    ids.insert(1003);
    ids.insert(1001); // 元素重复，自动忽略，不会报错
    ids.insert(2001);

    cout << "set大小: " << ids.size() << endl;
    cout << "元素（自动升序）: ";
    // 迭代器遍历，*it 取出迭代器指向的元素
    for (set<int>::iterator it = ids.begin(); it != ids.end(); ++it)
        cout << *it << " ";
    cout << endl;

    // ===== 2. set 查找 =====
    cout << "\n=== 查找 ===" << endl;
    set<int>::iterator found = ids.find(1002);
    if (found != ids.end())
        cout << "1002 存在" << endl;
    // count：存在返回1，不存在返回0（set元素唯一）
    cout << "9999 存在吗: " << (ids.count(9999) > 0 ? "是" : "否") << endl;

    // ===== 3. set经典用途：vector去重 =====
    cout << "\n=== set去重应用 ===" << endl;
    vector<int> raw = {5, 3, 8, 1, 5, 3, 9, 1, 2, 5, 8};
    cout << "原始vector: ";
    for (size_t i = 0; i < raw.size(); i++)
        cout << raw[i] << " ";
    cout << endl;

    // 直接用vector首尾迭代器构造set，自动完成【去重+排序】
    set<int> uniqueNums(raw.begin(), raw.end());
    cout << "set后: ";
    for (set<int>::iterator it = uniqueNums.begin(); it != uniqueNums.end(); ++it)
        cout << *it << " ";
    cout << " (去重后 " << uniqueNums.size() << " 个元素)" << endl;

    // ===== 4. 存储字符串set =====
    cout << "\n=== string set ===" << endl;
    set<string> deviceNames;
    deviceNames.insert("加热炉A");
    deviceNames.insert("冷却塔B");
    deviceNames.insert("反应釜C");
    deviceNames.insert("加热炉A"); // 重复字符串直接忽略

    cout << "设备名称（按字典序自动排序）:" << endl;
    for (set<string>::iterator it = deviceNames.begin(); it != deviceNames.end(); ++it)
    {
        cout << "  " << *it << endl;
    }

    // ===== 5. 修改排序规则：降序set =====
    // 默认 set<int> 等价 set<int, less<int>> 升序
    // greater<int> 指定比较规则，变成从大到小
    set<int, greater<int>> descNums;
    descNums.insert(10);
    descNums.insert(5);
    descNums.insert(20);
    descNums.insert(15);

    cout << "\n降序set: ";
    for (set<int, greater<int>>::iterator it = descNums.begin(); it != descNums.end(); ++it)
        cout << *it << " ";
    cout << endl;

    return 0;
}
```

输出结果：

```
set大小: 4
元素（自动升序）: 1001 1002 1003 2001

=== 查找 ===
1002 存在
9999 存在吗: 否

=== set去重应用 ===
原始vector: 5 3 8 1 5 3 9 1 2 5 8
set后: 1 2 3 5 8 9  (去重后 6 个元素)

=== string set ===
设备名称（按字典序自动排序）:
  反应釜C
  加热炉A
  冷却塔B

降序set: 20 15 10 5
```

---

## 3. multimap/multiset：允许重复键

multimap与map的唯一区别是key可以重复，其它都一样。multiset与set的唯一区别是元素可以重复。

```c++
#include <iostream>
#include <map>
#include <set>
#include <string>
using namespace std;

int main()
{
    // ===== multimap：允许同一个key对应多个value =====
    cout << "=== multimap ===" << endl;
    // key：设备名称，value：温度读数。一个设备可以有多条温度记录
    multimap<string, int> deviceReadings;

    // 同一个key可以多次插入，全部保留（map不允许重复key）
    deviceReadings.insert(make_pair("加热炉A", 75));
    deviceReadings.insert(make_pair("加热炉A", 78));
    deviceReadings.insert(make_pair("加热炉A", 82));
    deviceReadings.insert(make_pair("冷却塔B", 28));
    deviceReadings.insert(make_pair("冷却塔B", 29));
    deviceReadings.insert(make_pair("反应釜C", 120));

    cout << "共 " << deviceReadings.size() << " 条读数" << endl;
    cout << "加热炉A的读数数量: " << deviceReadings.count("加热炉A") << endl;

    // equal_range：获取这个key对应的【起始迭代器，结束迭代器】区间
    cout << "\n加热炉A的所有读数:" << endl;
    pair<multimap<string, int>::iterator,
        multimap<string, int>::iterator> range;
    range = deviceReadings.equal_range("加热炉A");
    // 遍历区间内所有同key数据
    for (multimap<string, int>::iterator it = range.first;
        it != range.second;
        ++it)
    {
        cout << "  " << it->first << " -> " << it->second << "度" << endl;
    }

    // ===== multiset：允许存储重复元素，自动升序 =====
    cout << "\n=== multiset ===" << endl;
    multiset<int> readings;
    readings.insert(75);
    readings.insert(80);
    readings.insert(75); // multiset允许重复元素，可以插入成功
    readings.insert(85);
    readings.insert(75);

    cout << "共 " << readings.size() << " 个读数" << endl;
    cout << "75出现的次数: " << readings.count(75) << endl;
    cout << "全部读数（自动排序）: ";
    for (multiset<int>::iterator it = readings.begin();
        it != readings.end();
        ++it)
        cout << *it << " ";
    cout << endl;

    // ⚠️重点：erase(值)，会删除容器里**所有等于该值**的元素
    readings.erase(75);
    cout << "erase(75)后剩 " << readings.size() << " 个:" << endl;
    for (multiset<int>::iterator it = readings.begin();
        it != readings.end();
        ++it)
        cout << *it << " ";
    cout << endl;

    return 0;
}
```

输出结果：

```
=== multimap ===
共 6 条读数
加热炉A的读数数量: 3

加热炉A的所有读数:
  加热炉A -> 75度
  加热炉A -> 78度
  加热炉A -> 82度

=== multiset ===
共 5 个读数
75出现的次数: 3
全部读数（自动排序）: 75 75 75 80 85
erase(75)后剩 2 个:
80 85
```

---

## 4. map自定义key类型

自定义类型作为map的key时，必须定义**比较运算符**（`operator<`）。

```c++
#include <iostream>
#include <map>
#include <string>
using namespace std;

// 复合键：车间+工位，作为map的key
struct Location
{
    int building;   // 车间号
    int station;    // 工位号
    string name;    // 位置名称

    Location(int b = 0, int s = 0, string n = "")
        : building(b), station(s), name(n) {}

    // ✅ 必须重载 < 运算符，map底层红黑树需要用它做排序、判断key相等
    bool operator<(const Location& other) const
    {
        // 先比较车间；车间相同再比较工位
        if (building != other.building)
            return building < other.building;
        return station < other.station;
    }
};

int main()
{
    // key是自定义结构体Location，value是温度
    map<Location, double> locationTemp;

    locationTemp[Location(1, 1, "1车间1号工位")] = 45.2;
    locationTemp[Location(1, 2, "1车间2号工位")] = 48.5;
    locationTemp[Location(2, 1, "2车间1号工位")] = 35.8;

    // 【重点】比较规则只看building、station，name不参与比较
    // 下面这个Location(1,1,"重复的") 和前面(1,1,"1车间1号工位")判定为同一个key，覆盖旧值
    locationTemp[Location(1, 1, "重复的")] = 99.9;

    cout << "位置温度（按车间号+工位号排序）:" << endl;
    for (map<Location, double>::iterator it = locationTemp.begin();
         it != locationTemp.end();
         ++it)
    {
        cout << "  车间" << it->first.building
             << "-工位" << it->first.station
             << " (" << it->first.name << "): "
             << it->second << "度" << endl;
    }

    // 查找：只要参与比较的字段(building,station)一致，就能找到，name随便填
    Location key(1, 2, "");
    map<Location, double>::iterator found = locationTemp.find(key);
    if (found != locationTemp.end())
        cout << "\n查找到 (1,2): " << found->second << "度" << endl;

    return 0;
}
```

输出结果：

```
位置温度（按车间号+工位号排序）:
  车间1-工位1 (1车间1号工位): 99.9度
  车间1-工位2 (1车间2号工位): 48.5度
  车间2-工位1 (2车间1号工位): 35.8度

查找到 (1,2): 48.5度
```

---

## 5. 迭代器进阶：类型、失效规则、auto

### 5.1 基础语法

```c++
#include <iostream>
#include <vector>
#include <map>
#include <string>
using namespace std;

int main()
{
    // ===== 1. 迭代器的3种常用类型 =====
    cout << "=== 迭代器类型 ===" << endl;
    vector<int> v;
    for (int i = 1; i <= 5; i++)
        v.push_back(i * 10);

    // 普通迭代器 iterator：可读、可修改容器元素
    cout << "普通迭代器: ";
    for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
    {
        *it = *it + 1; // 允许修改元素
        cout << *it << " ";
    }
    cout << endl;

    // const_iterator 常量迭代器：只读，不能修改元素
    cout << "const_iterator: ";
    for (vector<int>::const_iterator it = v.begin(); it != v.end(); ++it)
    {
        // *it = 100; // 编译报错！只读，禁止写
        cout << *it << " ";
    }
    cout << endl;

    // reverse_iterator 反向迭代器：从末尾向前遍历
    cout << "反向迭代器: ";
    for (vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++it)
    {
        cout << *it << " ";
    }
    cout << endl;

    // ===== 2. C++11 auto：自动推导迭代器类型，简化代码 =====
    cout << "\n=== auto简化迭代器 ===" << endl;
    map<string, int> stats;
    stats["正常"] = 85;
    stats["警告"] = 10;
    stats["报警"] = 5;

    // auto自动识别迭代器类型，不用手写很长的类型名
    cout << "auto遍历map: ";
    for (auto it = stats.begin(); it != stats.end(); ++it)
    {
        cout << it->first << "=" << it->second << " ";
    }
    cout << endl;

    // ===== 3. 迭代器失效（面试重点！） =====
    cout << "\n=== 迭代器失效演示 ===" << endl;
    cout << "vector迭代器失效规则：" << endl;
    cout << "  插入/删除：被操作位置之后的迭代器可能失效；"
            "扩容重新分配内存时，全部迭代器失效。" << endl;
    cout << "  erase(it)会返回下一个有效的迭代器，安全删除要用返回值\n" << endl;

    cout << "删除所有偶数前: ";
    for (size_t i = 0; i < v.size(); i++)
        cout << v[i] << " ";
    cout << endl;

    vector<int>::iterator it = v.begin();
    while (it != v.end())
    {
        if (*it % 2 == 0)
            it = v.erase(it); // erase返回下一个有效迭代器，接收返回值防止失效
        else
            ++it;
    }
    cout << "删除偶数后: ";
    for (size_t i = 0; i < v.size(); i++)
        cout << v[i] << " ";
    cout << endl;

    cout << "\nmap/set（红黑树容器）迭代器失效规则：" << endl;
    cout << "  插入：不会让任何迭代器失效；" << endl;
    cout << "  删除：仅被删掉元素的迭代器失效，其余迭代器保持有效\n" << endl;

    // map安全删除示例
    cout << "map安全删除（删除value>10的条目）:" << endl;
    auto mit = stats.begin();
    while (mit != stats.end())
    {
        if (mit->second > 10)
            mit = stats.erase(mit); // C++11后erase返回下一个迭代器
        else
            ++mit;
    }
    for (auto it = stats.begin(); it != stats.end(); ++it)
        cout << "  " << it->first << "=" << it->second << endl;

    return 0;
}
```

输出结果：

```
=== 迭代器类型 ===
普通迭代器: 11 21 31 41 51
const_iterator: 11 21 31 41 51
反向迭代器: 51 41 31 21 11

=== auto简化迭代器 ===
auto遍历map: 报警=5 警告=10 正常=85

=== 迭代器失效演示 ===
vector迭代器失效规则：
  插入/删除：被操作位置之后的迭代器可能失效；扩容重新分配内存时，全部迭代器失效。
  erase(it)会返回下一个有效的迭代器，安全删除要用返回值

删除所有偶数前: 11 21 31 41 51
删除偶数后: 11 21 31 41 51

map/set（红黑树容器）迭代器失效规则：
  插入：不会让任何迭代器失效；
  删除：仅被删掉元素的迭代器失效，其余迭代器保持有效

map安全删除（删除value>10的条目）:
  报警=5
  警告=10
```

### 5.2 面试重点：迭代器失效【高频面试考点】

> 1. vector（连续数组内存）：

- 插入元素：如果触发扩容，**全部迭代器失效**；不扩容，插入点后面迭代器失效；
- 删除元素：删除点之后所有迭代器失效；只有【被删除位置之后】的迭代器失效，删除位置之前的迭代器仍然有效；“删除点及之后的迭代器全部失效” 指的是：原来旧的那些迭代器（删除操作之前保存的迭代器）全部作废。`erase` 的返回值，是一个全新的，有指向被删元素下一个元素的有效迭代器，因此可以接着循环遍历vector<int> v。

- 安全写法：`it = v.erase(it);` 接收 erase 返回的新迭代器

> 如何理解删除元素呢？

```c++
vector<int> v = {10, 20, 30, 40, 50};
// it_old 指向 20
auto it_old = v.begin() + 1;
auto it_front = v.begin(); // 指向10（删除点前面）
auto it_back  = v.begin() + 2; // 指向30（删除点后面）

v.erase(it_old);
```

执行 erase 之后：

1. 数组变成 `{10,30,40,50}`，元素整体往前挪一格；

2. `it_front`（指向 10）：**仍然有效**（删除点前面，不受影响）；

3. `it_old`（原来指向 20）：**失效，不能再用**

4. `it_back`（原来指向 30）：**旧迭代器失效！**

5. `auto it_new = v.erase(it_old);`

   > `it_new` 是 erase**新构造出来**的迭代器，指向新数组里的 30，**这个新迭代器是合法可用的**

> 2. map /set/multimap /multiset（红黑树）

- 插入：**所有迭代器都不失效**
- 删除：**只有被删除的那个迭代器失效，其它迭代器完好**
- 同样推荐接收 erase 返回值，代码统一规范

vector：内存连续，增删容易导致迭代器大面积失效。

map/set：节点分散挂在树上，只有被删除节点的迭代器失效。

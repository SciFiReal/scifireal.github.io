---
title: "高级篇02-文件输入输出：fstream读写文本与二进制文件"

date: 2026-10-07T09:44:21+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇02-文件输入输出：fstream读写文本与二进制文件"

toc: true
---



> C++文件IO的核心类：
>
> - **ofstream**：Output File Stream，写文件（输出到文件）
> - **ifstream**：Input File Stream，读文件（从文件输入）
> - **fstream**：File Stream，可同时读写
>
> 它们都继承自`iostream`类家族，所以用法和cin/cout几乎一样——`<<` 写、`>>` 读、`getline` 读一行等。
>
> 文件IO的两个主要场景：
>
> 1. **文本文件**：人可读的格式。每行是一条记录，字段用分隔符分开。如配置文件、CSV、日志
> 2. **二进制文件**：机器可读的格式。直接读写内存中的字节表示，不做任何格式转换。如工业数据记录、图片、压缩文件

---

## 1. 写文本文件：ofstream

```c++
#include <iostream>
#include <fstream>   // 文件读写头文件
using namespace std;

int main()
{
    // 1. 创建文件写入流对象，打开文件
    // 默认模式：不存在则创建，存在则清空内容后写入
    ofstream outFile("sensor_data.txt");

    // 2. 必须判断文件是否打开成功（路径错误/权限不足会打开失败）
    if (!outFile.is_open())
    {
        cout << "无法打开文件进行写入！" << endl;
        return 1;
    }

    // 3. 写入CSV表头与固定传感器数据
    outFile << "设备ID,名称,温度,时间" << endl;
    outFile << "1001,加热炉A,78.5,2024-01-01 08:00:00" << endl;
    outFile << "1002,冷却塔B,28.3,2024-01-01 08:00:01" << endl;
    outFile << "2001,反应釜C,125.8,2024-01-01 08:00:02" << endl;

    // 4. 通过变量动态写入数据
    int deviceId = 3001;
    const char* name = "阀门D";
    double temperature = 45.2;
    outFile << deviceId << "," << name << "," << temperature
            << ",2024-01-01 08:00:03" << endl;

    // 5. 关闭文件（RAII机制会自动关闭，显式关闭更规范清晰）
    outFile.close();

    cout << "传感器数据写入成功！" << endl;

    return 0;
}
```

> 打开模式标记

```c++
ofstream f1("log.txt", ios::app);         // 追加写日志
ofstream f2("data.bin", ios::binary);     // 二进制写入
ifstream f3("config.txt", ios::in);       // 只读（默认）
fstream f4("data.dat", ios::in | ios::out | ios::binary); // 读写二进制
```

`ios::xxx` 是**打开文件的模式标记**，用 `|` 按位或来**组合多个模式**

> 常见的模式标记：

- `ios::out`：打开用于写入（默认，会清空已有内容）

- `ios::in`：打开用于读取

- `ios::app`：追加模式（Append），写在文件末尾

- `ios::trunc`：如果文件存在，截断为0字节（默认行为）

- `ios::binary`：二进制模式（不做换行符转换）

- `ios::ate`：打开后定位到文件末尾（At End）

---

## 2. 读文本文件：ifstream

```c++
#include <iostream>
#include <fstream>
#include <string>   // std::string、getline
using namespace std;

int main()
{
    // 1. 打开文件用于读取
    ifstream inFile("sensor_data.txt");
    if (!inFile.is_open())
    {
        cout << "无法打开文件读取!" << endl;
        return 1;
    }

    // ===== 方法1：按行读取（适合文本、日志，一次性读整行） =====
    cout << "===== 方法1：按行读取 =====" << endl;
    string line;
    int lineNum = 0;
    while (getline(inFile, line)) // 循环读一行，读到文件末尾EOF终止
    {
        lineNum++;
        cout << "第" << lineNum << "行: " << line << endl;
    }

    // 读完到末尾，流状态是eof，需要清空标志 + 移动读取指针，才能重新读
    inFile.clear();                // 清除eof错误标记
    inFile.seekg(0, ios::beg);     // 读取指针移回文件开头

    // ===== 方法2：按字段解析CSV（>> 自动跳过空格、换行） =====
    cout << "\n===== 方法2：按字段读取解析CSV =====" << endl;
    int id;
    string name;
    double temp;
    char comma;

    // 跳过表头第一行
    string header;
    getline(inFile, header);
    cout << "跳过标题行: " << header << endl;

    // 按逗号分割读取每一条传感器记录
    while (inFile >> id >> comma) // 读取设备ID + 逗号分隔符
    {
        getline(inFile, name, ','); // 读到逗号为止，取出名称
        inFile >> temp;
        inFile.ignore(100, '\n');   // 忽略本行剩下的内容直到换行

        cout << "设备: " << id << " " << name << " 温度: " << temp << endl;
    }

    // 判断结束原因
    if (inFile.eof())
        cout << "\n正常到达文件末尾" << endl;
    else
        cout << "\n读取过程中发生错误" << endl;

    inFile.close();
    return 0;
}
```

> **状态检查**：

- `is_open()`：文件是否成功打开
- `eof()`：是否到达文件末尾（End Of File）
- `fail()`：读取失败（如类型不匹配）
- `bad()`：严重错误（如磁盘错误）
- `good()`：一切正常
- `clear()`：清除错误状态，允许继续操作
- `seekg(pos)`：设置读指针位置（Get position）
- `seekp(pos)`：设置写指针位置（Put position）

---

## 3. 追加写入日志文件（工业日志工具类）

```c++
#include <iostream>
#include <fstream>
#include <ctime>
#include <string>
using namespace std;

// 工业日志工具类：追加模式写日志
class Logger
{
public:
    Logger(const char* filename)
        : logFileName(filename)
    {}

    void log(const char* level, const char* message)
    {
        // ios::app 追加模式：每次打开，内容写到文件末尾，不清空原有日志
        ofstream out(logFileName, ios::app);
        if (!out.is_open())
        {
            cerr << "日志文件打开失败！" << logFileName << endl;
            return;
        }

        // 获取当前系统时间
        time_t now = time(nullptr);
        char timeStr[30];
        // localtime 非线程安全！多线程环境禁止直接使用
        strftime(timeStr, sizeof(timeStr), "%Y-%m-%d %H:%M:%S", localtime(&now));

        out << "[" << timeStr << "] [" << level << "] " << message << endl;
        // out 局部对象，函数退出自动析构关闭文件，RAII
    }

private:
    string logFileName;
};

int main()
{
    Logger logger("system.log");
    logger.log("INFO", "系统启动");
    logger.log("INFO", "加载配置文件完成");
    logger.log("WARN", "设备1001温度接近阈值78.5度");
    logger.log("ERROR", "设备2001通信超时");
    logger.log("INFO", "系统正常运行");

    cout << "日志已写入 system.log\n" << endl;

    // ===== 读取日志演示 =====
    cout << "===== 读取日志文件 =====" << endl;
    ifstream in("system.log");
    if (in.is_open())
    {
        string line;
        while (getline(in, line))
        {
            cout << line << endl;
        }
        in.close();
    }
    else
    {
        cerr << "打开日志读取失败" << endl;
    }

    return 0;
}
```

输出：

```
[2026-10-07 16:20:10] [INFO] 系统启动
[2026-10-07 16:20:10] [INFO] 加载配置文件完成
[2026-10-07 16:20:10] [WARN] 设备1001温度接近阈值78.5度
[2026-10-07 16:20:10] [ERROR] 设备2001通信超时
[2026-10-07 16:20:10] [INFO] 系统正常运行
```

---

## 4. 读写二进制文件

二进制文件直接读写内存字节，不做格式转换。优点是体积小、读写快、精度不丢；缺点是不可读且平台相关（字节序/内存对齐不同）。

```c++
#include <iostream>
#include <fstream>
using namespace std;

// 工业传感器结构体，固定内存布局，适合二进制读写
struct SensorRecord
{
    int sensorId;
    double value;
    int timestamp;
    char status;  // 'N'正常，'A'报警，'E'错误
};

// 二进制写入：把结构体整块存入文件
void writeBinary(const char* filename)
{
    // ios::binary：二进制模式，不自动转换换行符
    ofstream out(filename, ios::binary);
    if (!out.is_open())
    {
        cerr << "无法创建二进制文件" << endl;
        return;
    }

    SensorRecord data[] = {
        {1001, 78.5, 1704067200, 'N'},
        {1002, 28.3, 1704067201, 'N'},
        {2001, 125.8, 1704067202, 'A'},
        {2002, 45.2, 1704067203, 'N'},
        {3001, 15.7, 1704067204, 'N'}
    };
    int count = sizeof(data) / sizeof(SensorRecord);

    // 先写记录数量（方便读的时候知道要读多少条）
    out.write(reinterpret_cast<char*>(&count), sizeof(int));
    // 整块写入结构体数组
    out.write(reinterpret_cast<char*>(data), sizeof(data));

    // out是局部RAII对象，函数结束自动关闭文件，close()可省略
    cout << "写入 " << count << " 条记录到 " << filename
         << "，文件大小: " << sizeof(int) + sizeof(data) << " 字节" << endl;
}

// 二进制读取：从文件读出结构体
void readBinary(const char* filename)
{
    ifstream in(filename, ios::binary);
    if (!in.is_open())
    {
        cerr << "无法打开二进制文件" << endl;
        return;
    }

    int count = 0;
    in.read(reinterpret_cast<char*>(&count), sizeof(int));
    cout << "文件包含 " << count << " 条记录" << endl;

    // 动态内存存放读到的记录
    SensorRecord* data = new SensorRecord[count];
    in.read(reinterpret_cast<char*>(data), count * sizeof(SensorRecord));

    streamsize bytesRead = in.gcount();
    cout << "本次实际读取 " << bytesRead << " 字节\n" << endl;

    for (int i = 0; i < count; i++)
    {
        cout << "记录" << i << ": ID=" << data[i].sensorId
             << " Value=" << data[i].value
             << " Time=" << data[i].timestamp
             << " Status=" << data[i].status << endl;
    }

    delete[] data; // 释放堆内存
}

int main()
{
    writeBinary("sensors.bin");
    cout << "\n--- 读取二进制文件 ---" << endl;
    readBinary("sensors.bin");
    return 0;
}
```

> **关键要点**：

- `write(char*, size)`：写size个字节到文件
- `read(char*, size)`：从文件读最多size个字节
- `gcount()`：返回上一次read实际读的字节数
- `reinterpret_cast<char*>(指针)`：把任意类型的指针强制解释为char*（字节指针）
- 二进制文件**不可跨平台**：不同编译器/CPU的struct对齐、字节序、double的表示可能不同。跨平台需要自定义序列化

---

## 5. 文件位置控制：随机访问

可以在文件中移动读写指针，实现跳读、覆盖写等操作：

```
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
    // ios::in|out：可读可写；binary：二进制；trunc：打开时清空旧内容
    fstream file("random.bin", ios::in | ios::out | ios::binary | ios::trunc);
    if (!file.is_open())
    {
        cout << "创建失败" << endl;
        return 1;
    }

    // 写入10个int
    int data[10];
    for (int i = 0; i < 10; ++i)
        data[i] = i * 100;
    file.write(reinterpret_cast<char*>(data), sizeof(data));

    // seekg：移动【读指针】
    file.seekg(4 * sizeof(int));   // 跳到索引4（第5个元素）
    int val;
    file.read(reinterpret_cast<char*>(&val), sizeof(int));
    cout << "第5条数据 = " << val << endl; // 输出400

    // seekp：移动【写指针】，覆盖写入
    file.seekp(2 * sizeof(int));   // 跳到索引2（第3个元素）
    int newVal = 9999;
    file.write(reinterpret_cast<char*>(&newVal), sizeof(int));

    // 读全部数据验证
    file.seekg(0);                 // 读指针移回文件开头
    int result[10];
    file.read(reinterpret_cast<char*>(result), sizeof(result));
    cout << "全部数据: ";
    for (int i = 0; i < 10; ++i)
        cout << result[i] << " ";
    cout << endl;

    // 从文件末尾向前偏移读取
    // ios::beg 起点开头；ios::cur 当前位置；ios::end 文件末尾
    file.seekg(-2 * (int)sizeof(int), ios::end);
    file.read(reinterpret_cast<char*>(&val), sizeof(int));
    cout << "倒数第2条 = " << val << endl;

    return 0;
}
```

输出结果：

```
第5条数据 = 400
全部数据: 0 100 9999 300 400 500 600 700 800 900 
倒数第2条 = 800
```

> 核心知识点

- `fstream`：同时支持读、写。
- `seekg(pos)`：**g = get，移动读指针**，用来定位读取位置。
- `seekp(pos)`：**p = put，移动写指针**，用来定位写入位置。
- 偏移基准：
  - `ios::beg`：从文件开头算偏移（默认）
  - `ios::cur`：从当前指针位置算偏移
  - `ios::end`：从文件末尾算偏移（偏移量一般写负数，向前找）
- `ios::trunc`：打开文件直接清空原有内容。

---

## 6. 工业场景实战：配置文件读写
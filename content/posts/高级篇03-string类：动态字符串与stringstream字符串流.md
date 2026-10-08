---
title: "高级篇03-string类：动态字符串与stringstream字符串流"

date: 2026-10-07T10:10:57+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "高级篇03-string类：动态字符串与stringstream字符串流"

toc: true
---



> 在基础篇学过C风格字符串——字符数组，以'\0'结尾。它灵活但易错：固定大小（易越界）、手动管理、函数名不直观（strcmp/strcpy/strcat）。C++标准库提供了 `std::string` 类——动态大小、自动内存管理、丰富的成员方法。

---

## 1. string基本用法

```c++
#include <iostream>
#include <string>   // C++ std::string 字符串类必备头文件
#include <cstring>  // C语言字符串函数 strcpy
#include <stdexcept>// out_of_range 异常头文件

using namespace std;

int main()
{
    // ====================== 1. string 的多种初始化方式 ======================
    cout << "========== 1. string 多种初始化 ==========\n";
    string s1;                      // 方式1：默认构造，空字符串
    string s2("Hello, C++!");       // 方式2：用C风格字符串初始化
    string s3 = "工业字符串";        // 方式3：赋值写法，可读性高
    string s4(10, 'A');             // 方式4：生成10个'A'组成的字符串
    string s5(s2);                  // 方式5：拷贝构造，复制s2的内容
    string s6(s2, 7, 3);            // 方式6：截取子串
                                    // 参数：(源字符串, 起始下标, 截取字符个数)
                                    // s2第7号下标开始，取3个字符 → "C++"

    cout << "s1 = [" << s1 << "] (长度=" << s1.size() << ")\n";
    cout << "s2 = [" << s2 << "] (长度=" << s2.size() << ")\n";
    cout << "s3 = [" << s3 << "]\n";
    cout << "s4 = [" << s4 << "]\n";
    cout << "s5 = [" << s5 << "]\n";
    cout << "s6 = [" << s6 << "]\n";


    // ====================== 2. 字符串输入输出 ======================
    cout << "\n========== 2. 字符串输入输出 ==========\n";
    string input;
    cout << "请输入不带空格的字符串(用cin >>): ";
    cin >> input;
    cout << "cin读取结果：" << input << "\n";
    // ✅ 重点：cin >> 遇到空格、回车就停止读取，不能读带空格句子

    cin.ignore(); // 清空缓冲区残留换行，防止getline直接读空
    cout << "请输入带空格的字符串(用getline): ";
    getline(cin, input); // getline：读取一整行，可以包含空格
    cout << "getline读取结果：" << input << "\n";


    // ====================== 3. 获取字符串基础属性 ======================
    cout << "\n========== 3. 字符串基础属性 ==========\n";
    cout << "字符串 s2 = \"" << s2 << "\"\n";
    // size() 和 length() 完全等价，都是返回字符个数
    cout << "  字符长度 size() = " << s2.size() << "\n";
    cout << "  字符长度 length() = " << s2.length() << "\n";
    cout << "  s1是否为空 empty()：" << (s1.empty() ? "是" : "否") << "\n";
    cout << "  理论最大可存放字符 max_size()：" << s2.max_size() << "\n";


    // ====================== 4. 访问字符串里单个字符 ======================
    cout << "\n========== 4. 访问单个字符 [] 和 at() ==========\n";
    cout << "遍历 s2 的每个字符：\n";
    for (size_t i = 0; i < s2.size(); i++)
    {
        cout << "  s2[" << i << "] = '" << s2[i] << "'";
        if (i == s2.size() - 1)
            cout << "  <- 最后一个字符";
        cout << "\n";
    }
    /*
     * []：下标访问，速度快，但**不检查越界**。如果下标超出范围，程序直接崩溃
     * at()：安全访问，会检查下标。越界会抛出 out_of_range 异常，可以 try-catch 捕获
     */
    try
    {
        cout << "尝试访问 s2.at(100)：";
        cout << s2.at(100) << "\n";
    }
    catch (const out_of_range& e)
    {
        cout << "捕获越界异常：" << e.what() << "\n";
    }


    // ====================== 5. 字符串比较 ======================
    cout << "\n========== 5. 字符串比较 ==========\n";
    // C++ string 可以直接用 == != > < >= <= 比较！
    // 底层按 ASCII 字典序比较，区分大小写，不用C语言strcmp
    string a = "apple", b = "banana";
    cout << "apple == banana ? " << (a == b ? "是" : "否") << "\n";
    cout << "apple < banana  ? " << (a < b ? "是" : "否") << " （字典序）\n";
    cout << "apple > Apple   ? " << (a > "Apple" ? "是" : "否") << " （大写字母ASCII更小）\n";


    // ====================== 6. 字符串拼接、追加 ======================
    cout << "\n========== 6. 字符串拼接 ==========\n";
    string msg = "设备";
    msg += "温度";   // += 向后追加字符串
    msg += ':';     // += 也可以追加单个字符
    msg += ' ';
    msg = msg + " 78.5"; // + 运算符：两个string拼接，生成新字符串
    cout << "拼接结果：" << msg << "\n";


    // ====================== 7. C++ string 和 C风格 char* 互转 ======================
    cout << "\n========== 7. string 和 C字符串互转 ==========\n";
    // c_str()：返回 const char*，兼容C语言函数（strcpy、printf等）
    cout << "s2.c_str() -> C风格字符串：" << s2.c_str() << "\n";
    char buf[100];
    strcpy(buf, s2.c_str()); // 把string复制到char字符数组
    cout << "拷贝到char数组buf：" << buf << "\n";

    return 0;
}
```

输出示例：

```
========== 1. string 多种初始化 ==========
s1 = [] (长度=0)
s2 = [Hello, C++!] (长度=11)
s3 = [工业字符串]
s4 = [AAAAAAAAAA]
s5 = [Hello, C++!]
s6 = [C++]

========== 2. 字符串输入输出 ==========
请输入不带空格的字符串(用cin >>): hello
cin读取结果：hello
请输入带空格的字符串(用getline): hello world
getline读取结果：hello world

========== 3. 字符串基础属性 ==========
字符串 s2 = "Hello, C++!"
  字符长度 size() = 11
  字符长度 length() = 11
  s1是否为空 empty()：是
  理论最大可存放字符 max_size()：4611686018427387903

========== 4. 访问单个字符 [] 和 at() ==========
遍历 s2 的每个字符：
  s2[0] = 'H'
  s2[1] = 'e'
  s2[2] = 'l'
  s2[3] = 'l'
  s2[4] = 'o'
  s2[5] = ','
  s2[6] = ' '
  s2[7] = 'C'
  s2[8] = '+'
  s2[9] = '+'
  s2[10] = '!'  <- 最后一个字符
尝试访问 s2.at(100)：捕获越界异常：basic_string::at

========== 5. 字符串比较 ==========
apple == banana ? 否
apple < banana  ? 是 （字典序）
apple > Apple   ? 是 （大写字母ASCII更小）

========== 6. 字符串拼接 ==========
拼接结果：设备: 78.5

========== 7. string 和 C字符串互转 ==========
s2.c_str() -> C风格字符串：Hello, C++!
拷贝到char数组buf：Hello, C++!
```

> 1. 初始化：日常写`string s1 = xxx：`
> 2. 输入：`cin >> s`：读到空格停止，适合单个单词；`getline(cin，s)`：读取一整行，包括空格，适合句子。
> 3. 长度：`size()`和`length()`一模一样，推荐些`size()`（STL容器统一接口）
> 4. 字符访问：`s[i]`，快，不检查越界；`s.at(i)`，取出下标为i的那个数，安全，检查越界
> 5. 比较：`== < >`直接比较大小，不用`strcmp`，这是C风格的字符串比较函数
> 6. 拼接：`+`拼接；`+= `追加
> 7. `empty()` 判断是否为空，等价于`s.size() == 0`

---

## 2. string高级知识

```c++
#include <iostream>
#include <string>
using namespace std;

int main()
{
    // 原始字符串：模拟工业设备上报的一行文本，key=value用逗号分隔
    string data = "设备ID=1001,名称=加热炉A,温度=78.5,状态=运行,位置=车间1";
    cout << "===== C++ string 常用高级操作 =====\n";
    cout << "原始字符串：" << data << "\n\n";

    // ===================== 1. 查找 find / rfind / find_first_of =====================
    cout << "---------- 1. 字符串查找 ----------\n";
    // find(子串)：从左向右查找子串，返回起始下标；找不到返回 string::npos
    size_t pos = data.find("温度");
    if (pos != string::npos)
    {
        cout << "find(\"温度\")：\"温度\"首次出现下标 = " << pos << "\n";
    }
    else
    {
        cout << "没有找到【温度】\n";
    }

    // rfind(子串)：从右向左反向查找，找最后一次出现的位置
    pos = data.rfind("=");
    if (pos != string::npos)
    {
        cout << "rfind(\"=\")：最后一个等号下标 = " << pos << "\n";
    }

    // find_first_of(字符集)：查找【集合中任意一个字符】第一次出现的位置
    pos = data.find_first_of(",");
    cout << "find_first_of(\",\")：第一个逗号下标 = " << pos << "\n\n";


    // ===================== 2. 截取子串 substr =====================
    cout << "---------- 2. 截取子串 substr ----------\n";
    // substr(起始下标, 截取字符个数)
    // data.find("温度=")找到"温度="起始位置，+6 跳过"温度="这三个字
    string temp = data.substr(data.find("温度=") + 6, 4);
    cout << "提取温度值：" << temp << "\n\n";


    // ===================== 3. 替换 replace =====================
    cout << "---------- 3. 替换 replace ----------\n";
    string dataCopy = data; // 复制一份，不破坏原始字符串
    size_t statusPos = dataCopy.find("状态");
    if (statusPos != string::npos)
    {
        // 找到"状态"后面的等号
        size_t start = dataCopy.find('=', statusPos) + 1;
        // 从start位置往后找逗号，就是当前这一段的结束位置
        size_t end = dataCopy.find(',', start);
        // replace(起始位置, 需要替换多少个字符, 新字符串)
        dataCopy.replace(start, end - start, "停机");
        cout << "替换【状态=运行】→【状态=停机】：\n" << dataCopy << "\n\n";
    }


    // ===================== 4. 插入 insert =====================
    cout << "---------- 4. 插入 insert ----------\n";
    // insert(插入位置, 待插入字符串)：在pos前面插入内容
    dataCopy.insert(0, "[设备数据] ");
    cout << "在最前面插入标记：" << dataCopy << "\n\n";


    // ===================== 5. 删除 erase =====================
    cout << "---------- 5. 删除 erase ----------\n";
    // erase(起始位置, 删除字符数量)
    dataCopy.erase(0, 7); // 从下标0开始，删掉6个字符 "[设备数据]"
    cout << "删除开头6个字符：" << dataCopy << "\n\n";


    // ===================== 6. 清空 clear =====================
    cout << "---------- 6. 清空 clear ----------\n";
    dataCopy.clear();
    cout << "clear()清空字符串：内容='" << dataCopy << "'，长度=" << dataCopy.size() << "\n\n";


    // ===================== 7. 综合实战：分割逗号分隔的 key=value =====================
    cout << "---------- 7. 实战：分割逗号字符串，解析key=value ----------\n";
    string s = data;
    // 循环：字符串不为空就持续切割
    while (!s.empty())
    {
        size_t comma = s.find(',');
        string token;
        if (comma == string::npos)
        {
            // 找不到逗号，说明是最后一段
            token = s;
            s.clear();
        }
        else
        {
            // 截取逗号前面一段；剩余字符串更新为逗号后面内容
            token = s.substr(0, comma);
            s = s.substr(comma + 1);
        }
        // 对这一段 "key=value"，再按等号拆分
        size_t eq = token.find('=');
        string key = token.substr(0, eq);
        string value = token.substr(eq + 1);
        cout << "  " << key << " = " << value << "\n";
    }

    return 0;
}
```

输出示例：

```
===== C++ string 常用高级操作 =====
原始字符串：设备ID=1001,名称=加热炉A,温度=78.5,状态=运行,位置=车间1

---------- 1. 字符串查找 ----------
find("温度")："温度"首次出现下标 = 25
rfind("=")：最后一个等号下标 = 49
find_first_of(",")：第一个逗号下标 = 11

---------- 2. 截取子串 substr ----------
提取温度值：8.5,

---------- 3. 替换 replace ----------
替换【状态=运行】→【状态=停机】：
设备ID=1001,名称=加热炉A,温度=78.5,状态=停机,位置=车间1

---------- 4. 插入 insert ----------
在最前面插入标记：[设备数据] 设备ID=1001,名称=加热炉A,温度=78.5,状态=停机,位置=车间1

---------- 5. 删除 erase ----------
删除开头6个字符：据] 设备ID=1001,名称=加热炉A,温度=78.5,状态=停机,位置=车间1

---------- 6. 清空 clear ----------
clear()清空字符串：内容=''，长度=0

---------- 7. 实战：分割逗号字符串，解析key=value ----------
  设备ID = 1001
  名称 = 加热炉A
  温度 = 78.5
  状态 = 运行
  位置 = 车间1
```

> 1. 字符串查找：windows下编译器是`GBK` 编码；一个汉字占用2格字节；数字和字母，其它符号占一个字节。`UTF-8`编码，一个汉字占3个字节，数组和字母，其它符号占一个字节。
> 1. string根本不认`gbk`还是`utf-8`，只认一连串的字节。
> 1. `string::npos：find`找不到目标的时候，放回一个特殊值`string::npos`，代表不存在该位置。类型是`size_t`（无符号整数），判断查找结果必须些；无符号整数类型的别名（typedef）永远是大于等于0。
> 1. 综合实战：分割逗号分隔的 `key=value`；C++标准库中没有自带`split()`分割函数，所以工业场景中常用这套逻辑。手写字符串分割代码。

---

## 3. stringstream：字符串与其他类型互转

```c++
#include <iostream>
#include <sstream>   // stringstream、istringstream、ostringstream 头文件
#include <string>
using namespace std;

int main()
{
    // ======================================
    // 1. ostringstream：数值 → 字符串（拼接）
    // ======================================
    cout << "===== 1. ostringstream：数值拼接成字符串 =====" << endl;
    // ostringstream：输出字符串流，专门用来【把各种数据写入，拼成一个大字符串】
    ostringstream oss;
    int id = 1001;
    double temp = 78.5;
    string status = "正常";

    // 用法和 cout 一模一样！直接 << 把数字、字符串丢进去自动拼接
    oss << "设备" << id << ": 温度=" << temp << "度, 状态=" << status;
    // oss.str()：取出流里面拼接好的完整字符串
    string result = oss.str();
    cout << result << "\n\n";


    // ======================================
    // 2. istringstream：字符串 → 解析数值
    // ======================================
    cout << "===== 2. istringstream：字符串拆成多个变量 =====" << endl;
    // istringstream：输入字符串流。把一段字符串，当成控制台输入，用 >> 读取
    string data = "1001 78.5 1.5 正常";
    istringstream iss(data);

    int parsedId;
    double parsedTemp;
    double parsedPressure;
    string parsedStatus;

    // 和 cin >> 用法完全一样！>> 默认自动按空格分割，自动类型转换
    iss >> parsedId >> parsedTemp >> parsedPressure >> parsedStatus;
    cout << "解析结果: ID=" << parsedId
         << " 温度=" << parsedTemp
         << " 压力=" << parsedPressure
         << " 状态=" << parsedStatus << "\n\n";


    // ======================================
    // 3. 实战：解析CSV逗号分隔文本（工业场景非常常用）
    // ======================================
    cout << "===== 3. 实战：CSV逗号分隔解析（getline） =====" << endl;
    string csv = "1001,加热炉A,78.5,运行\n"
                 "1002,冷却塔B,28.3,运行\n"
                 "2001,反应釜C,125.8,报警";
    istringstream csvStream(csv);
    string line;

    // getline(流, 字符串)：按换行符 \n 读取，一次读一整行
    while (getline(csvStream, line))
    {
        // 取出一行后，再放到新的istringstream，按逗号分割每个字段
        istringstream lineStream(line);
        string field;
        int fieldIdx = 0;

        // getline第三个参数：getline(stream, 变量, 分隔符)
        // 代表：读到【逗号】就停止，逗号不存入field
        while (getline(lineStream, field, ','))
        {
            if(fieldIdx > 0) cout << " | ";
            cout << field;
            fieldIdx++;
        }
        cout << endl;
    }
    cout << endl;


    // ======================================
    // 4. 解析失败判断：fail() 检测转换是否出错
    // ======================================
    cout << "===== 4. 解析失败检测 fail() =====" << endl;
    string badData = "这不是数字";
    istringstream badStream(badData);
    int x;
    badStream >> x;   // 尝试把中文读到int变量，肯定失败

    // .fail()：如果上一次读取/类型转换失败，返回true
    if (badStream.fail())
        cout << "解析 '" << badData << "' 为整数失败（正确）\n\n";


    // ======================================
    // 5. stringstream：可读写，流的复用（重点坑点）
    // ======================================
    cout << "===== 5. stringstream 读写复用 =====" << endl;
    // stringstream：既能读 >>，也能写 <<，是上面两个的合体
    stringstream ss;

    ss << "42";        // 往流写入文本
    int num;
    ss >> num;         // 从流读出，转成int
    cout << "第一次：写入'42'，读出整数 = " << num << endl;

    // ✅ 复用流必须两步！顺序不能乱
    ss.clear();        // 清除流的错误标记（eof/fail标志）
    ss.str("");        // 清空流里面保存的字符串内容
    ss << "3.14";
    double pi;
    ss >> pi;
    cout << "清空复用后：写入'3.14'，读出double = " << pi << endl;

    return 0;
}
```

输出结果：

```
===== 1. ostringstream：数值拼接成字符串 =====
设备1001: 温度=78.5度, 状态=正常

===== 2. istringstream：字符串拆成多个变量 =====
解析结果: ID=1001 温度=78.5 压力=1.5 状态=正常

===== 3. 实战：CSV逗号分隔解析（getline） =====
1001 | 加热炉A | 78.5 | 运行
1002 | 冷却塔B | 28.3 | 运行
2001 | 反应釜C | 125.8 | 报警

===== 4. 解析失败检测 fail() =====
解析 '这不是数字' 为整数失败（正确）

===== 5. stringstream 读写复用 =====
第一次：写入'42'，读出整数 = 42
清空复用后：写入'3.14'，读出double = 3.14
```

> 1. 三大流的区分

`ostringstream`：输出字符串流：数字、变量拼成字符串，内存数据转向字符串；

`istringstream`：输入字符串流：字符串拆分，提取数字，字符串转向内存变量；

`stringstream`：双向流，上面两个合并，又读又写；

> 2. `getline`两种用法

- `getline(stream, str)`：默认按换行符`\n`分割，读取一整行；
- `getline(stream, str, ch)`：自定义分隔符，读到字符ch就停止；

> 3. 流复用的大坑

```
ss.clear();
ss.str("");
```

- `clear()`：**重置流状态标志**（eof 到达末尾、fail 失败标记）。一旦流读取失败，标记会永久锁死，后面再读全部失效，必须 clear 清除标记。
- `ss.str("")`：**清空流里面存储的字符串内容**。 顺序：**先 clear，再 str ("")**。

> 4. 错误判断 `.fail()`

当 `>>` 读取失败（文本格式不对，比如中文读 int），流内部会打上失败标记，`.fail()`返回 true。 配套还有：

- `.eof()`：是否读到末尾
- `.good()`：流状态正常，无错误

---

`stringstream`的核心价值：

- 类型转换：int/double/string 互转
- 字符串拼接：多个混合类型拼接，比`+=`更自然
- 解析字符串：从字符串中按空白/分隔符提取数据
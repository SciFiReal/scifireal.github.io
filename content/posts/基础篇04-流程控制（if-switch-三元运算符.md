---
title: "基础篇04-流程控制（if-switch）"

date: 2026-10-05T00:07:12+08:00

draft: false

categories: ["C++"]

tags: ["C++"]

summary: "基础篇04-流程控制（if-switch）"

toc: true
---

## 1. 分支结构

if-else、switch-case是三种基础流程结构中的分支结构，根据条件真假，选择性执行对应代码。另外两种如下

- 顺序结构：代码自上而下一次执行，是默认的基础流程
- 循环结构：重复执行代码，直到不满足循环条件，跳出循环

所有C++程序的运行逻辑，都基于上述三种基础流程结构。

### 1.1 if-else 

- 单分支if：满足条件执行，不满足不执行，适合单次判断

  ```c++
  bool comparisionResult = 3 == 3 ;
  if (comparisionResult)  // true，执行
  {
    std::cout << "Hello World!,this is true" << std::endl;
  }
  ```

- 双分支if-else：二选一执行

  ```c++
  bool comparisionResult = 3 == 7 ;
  if (comparisionResult)   // true
  {
    std::cout << "Hello World!,this is true" << std::endl;
  }
  else     // false
  {
    std::cout << "Hello World!,this is false" << std::endl;
  }
  ```

- 多分支if-else if-else：多级区间判断，适合多区间判断

  ```c++
  double score =  89.4;
  const double EPS = 1e-6;  // double的比较需要加EPS
  	
    if (score > 90.0 + EPS ) 
    {
    	std::cout << "优秀" << std::endl;
    }
    else if(score > 70.0 + EPS)
    {
    	std::cout << "良好" << std::endl;  // 良好
    }
    else if (score > 60.0 + EPS)
    {
    	std::cout << "及格" << std::endl;
    }
    else
    {
    	std::cout << "不及格" << std::endl;
    } 
  ```

  ==注意事项==

  > 1. else的就近匹配原则：就近if，无视缩进；想改变绑定，必须加{}
  > 2. if/else必须带大括号，增加阅读体验感
  > 3. 多分枝判断优先级要从高到低
  > 4. 浮点判断必须叠加误差阈值EPS
  > 5. 禁止三层嵌套判断

---

### 1.2 switch-case

专门用于固定离散数值匹配。

- switch仅支持整型、字符型、枚举离散值，不支持浮点、字符串
- case为精准等值匹配，无法做区间判断
- default为兜底分支，防止参数越界

注意事项：case击穿问题

```c++
int cmdCode = 1;

switch(cmdCode) 
{
	case 1:
		std::cout << "设备启动" << std::endl;
	case 2:
  	std::cout << "设备暂停" << std::endl;
  	break;
  case 3:
		std::cout << "设备复位" << std::endl;
		break;
	default:
		std::cout << "指令失效" << std::endl;
		break;
}
```

输出：

```bash
设备启动
设备暂停
```



标准写法：每个case都对应一个break；必须带有default；如果多个指令发出相同效果，可以合并case，如下所示：

```c++
//  多个指令触发设备休眠
case 4:
case 5:
case 6:
	std::cout << "设备进入休眠模式" << std::endl;
	break;	
```


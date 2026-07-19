---
title: C#学习【进阶篇】
published: 2026-07-17
pinned: false
description: "此文档记录了C#学习的学习内容"
image: "./Csharp_Image/Basic.avif"
tags: ["编程","C#"]
category: 学习记录
slug: Csharp_P2
---


# C# 进阶篇

> 本篇覆盖复杂数据类型、函数和排序，学完能写出结构化的程序。

---

## 一、枚举（enum）

枚举就是给一组相关的数值起个名字，让代码更可读。比如季节、星期、状态码，用数字记容易乱，用枚举一眼就懂。

### 1. 基本定义

```csharp
// 定义枚举，底层是 int，从 0 开始
enum Season { Spring, Summer, Autumn, Winter }

// Spring=0, Summer=1, Autumn=2, Winter=3

Season s = Season.Summer;
Console.WriteLine(s);        // 输出: Summer
Console.WriteLine((int)s);   // 输出: 1
```

### 2. 自定义编号

```csharp
enum HttpStatus { Ok = 200, NotFound = 404, ServerError = 500 }

HttpStatus code = HttpStatus.NotFound;
Console.WriteLine((int)code); // 输出: 404
```

### 3. 枚举与其它类型转换

```csharp
enum Season { Spring, Summer, Autumn, Winter }

Season s = Season.Autumn;

// 枚举 -> int（强转）
int n = (int)s;
Console.WriteLine(n);  // 输出: 2

// 枚举 -> string
string str = s.ToString();
Console.WriteLine(str); // 输出: Autumn

// string -> 枚举（Enum.Parse，找不到会抛异常）
Season s2 = (Season)Enum.Parse(typeof(Season), "Winter");
Console.WriteLine(s2);  // 输出: Winter

// string -> 枚举（TryParse，安全写法）
bool ok = Enum.TryParse<Season>("Spring", out Season s3);
Console.WriteLine(ok + " " + s3); // 输出: True Spring
```

### 4. 常用方法

```csharp
enum Season { Spring, Summer, Autumn, Winter }

// 获取所有枚举值
foreach (Season item in Enum.GetValues(typeof(Season)))
{
    Console.WriteLine(item);
}
// 输出:
// Spring
// Summer
// Autumn
// Winter

// 判断某值是否是合法枚举
Console.WriteLine(Enum.IsDefined(typeof(Season), 3));   // 输出: True
Console.WriteLine(Enum.IsDefined(typeof(Season), 99));  // 输出: False
```

### 5. [Flags] 标志枚举

用于权限等可组合的场景，值必须是 2 的幂（1, 2, 4, 8...）。

```csharp
[Flags]
enum Permission { Read = 1, Write = 2, Execute = 4 }

// 用 | 组合权限
Permission p = Permission.Read | Permission.Write;
Console.WriteLine(p); // 输出: Read, Write

// 用 & 判断是否包含某权限
bool canWrite = (p & Permission.Write) == Permission.Write;
Console.WriteLine(canWrite); // 输出: True
```

### 6. 枚举 + switch 黄金搭档

```csharp
enum Season { Spring, Summer, Autumn, Winter }

void Show(Season s)
{
    switch (s)
    {
        case Season.Spring:
            Console.WriteLine("春暖花开");
            break;
        case Season.Summer:
            Console.WriteLine("烈日炎炎");
            break;
        case Season.Autumn:
            Console.WriteLine("秋高气爽");
            break;
        case Season.Winter:
            Console.WriteLine("寒风刺骨");
            break;
        default:
            Console.WriteLine("未知季节");
            break;
    }
}

Show(Season.Summer); // 输出: 烈日炎炎
```

### ⚠️ 常见坑

```csharp
enum Season { Spring, Summer, Autumn, Winter }

// 坑：int 超出范围转枚举不会报错！
Season s = (Season)999;
Console.WriteLine(s); // 输出: 999（不报错，但不是合法枚举值）

// 所以用前最好用 Enum.IsDefined 检查
```

---

## 二、数组

数组是同一类型数据的固定长度集合。

### 1. 一维数组

#### 声明初始化的四种方式

```csharp
// 方式1：声明 + 指定长度
int[] a1 = new int[3];          // {0, 0, 0}

// 方式2：声明 + 直接给值
int[] a2 = new int[] { 1, 2, 3 };

// 方式3：简写（省略 new int[]）
int[] a3 = { 1, 2, 3 };

// 方式4：先声明后赋值
int[] a4;
a4 = new int[] { 1, 2, 3 };
```

#### 访问与遍历

```csharp
int[] arr = { 10, 20, 30 };

// 访问：下标范围 0 ~ Length-1
Console.WriteLine(arr[0]);      // 输出: 10
Console.WriteLine(arr.Length);  // 输出: 3

// for 遍历：能修改元素
for (int i = 0; i < arr.Length; i++)
{
    arr[i] *= 2;
}
// arr 变成 {20, 40, 60}

// foreach 遍历：只读，不能改
foreach (int x in arr)
{
    Console.WriteLine(x);
}
// 输出:
// 20
// 40
// 60
```

#### 常用方法

```csharp
int[] arr = { 5, 3, 8, 1, 9 };

Array.Sort(arr);            // 升序排序，arr 变成 {1, 3, 5, 8, 9}
Array.Reverse(arr);         // 反转，arr 变成 {9, 8, 5, 3, 1}

int idx = Array.IndexOf(arr, 5); // 输出: 2（找不到返回 -1）
Console.WriteLine(idx);

int[] copy = new int[3];
Array.Copy(arr, copy, 3);   // 拷贝前3个，copy = {9, 8, 5}

int[] cloned = (int[])arr.Clone(); // 浅拷贝一份
```

#### ⚠️ 常见坑

```csharp
// 坑1：数组是引用类型，直接赋值不是复制
int[] a = { 1, 2, 3 };
int[] b = a;
b[0] = 99;
Console.WriteLine(a[0]); // 输出: 99（a 也被改了！）

// 正确做法
int[] c = (int[])a.Clone();
c[0] = 0;
Console.WriteLine(a[0]); // 输出: 99（a 不受影响）

// 坑2：简写 { } 只能在声明时用
// int[] d;
// d = { 1, 2, 3 };  // 编译错误！
int[] d;
d = new int[] { 1, 2, 3 }; // 正确

// 坑3：数组长度固定，动态增删用 List
// arr.Add(4);  // 编译错误，数组没有 Add 方法
```

### 2. 二维数组

每行每列都一样长的"矩形"数组。

```csharp
// 声明：3行4列
int[,] arr = new int[3, 4];

// 声明并初始化
int[,] matrix = {
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

// 访问：arr[行, 列]（注意是一个方括号里用逗号）
Console.WriteLine(matrix[1, 2]); // 输出: 7

// 行数：GetLength(0)
Console.WriteLine(matrix.GetLength(0)); // 输出: 3
// 列数：GetLength(1)
Console.WriteLine(matrix.GetLength(1)); // 输出: 4
// Length 是总元素数（行×列）
Console.WriteLine(matrix.Length); // 输出: 12

// 遍历：嵌套 for
for (int i = 0; i < matrix.GetLength(0); i++)
{
    for (int j = 0; j < matrix.GetLength(1); j++)
    {
        Console.Write(matrix[i, j] + " ");
    }
    Console.WriteLine();
}
// 输出:
// 1 2 3 4
// 5 6 7 8
// 9 10 11 12
```

#### ⚠️ 常见坑

```csharp
// 坑：二维数组每行列数必须相同，不能像交错数组那样不规则
// int[,] wrong = { { 1, 2 }, { 3, 4, 5 } }; // 编译错误
```

### 3. 交错数组

数组的数组，每行可以长度不同。

```csharp
// 声明：3行，第二个括号留空
int[][] arr = new int[3][];

// 每行单独 new
arr[0] = new int[] { 1, 2 };
arr[1] = new int[] { 3, 4, 5, 6 };
arr[2] = new int[] { 7, 8, 9 };

// 访问：arr[行][列]（两个方括号）
Console.WriteLine(arr[1][2]); // 输出: 5

// 行数：arr.Length
Console.WriteLine(arr.Length);       // 输出: 3
// 第 i 行的列数：arr[i].Length
Console.WriteLine(arr[1].Length);    // 输出: 4

// 遍历
for (int i = 0; i < arr.Length; i++)
{
    for (int j = 0; j < arr[i].Length; j++)
    {
        Console.Write(arr[i][j] + " ");
    }
    Console.WriteLine();
}
// 输出:
// 1 2
// 3 4 5 6
// 7 8 9
```

#### 二维数组 vs 交错数组 对比表

| 对比项 | 二维数组 `int[,]` | 交错数组 `int[][]` |
|--------|------------------|-------------------|
| 语法 | `arr[行, 列]`（一对方括号） | `arr[行][列]`（两对方括号） |
| 每行长度 | 必须相同 | 可以不同 |
| 声明 | `new int[3, 4]` | `new int[3][]`，每行单独 new |
| 行数 | `GetLength(0)` | `Length` |
| 列数 | `GetLength(1)` | `arr[i].Length` |
| 内存 | 连续存储 | 不连续 |
| 使用场景 | 表格、矩阵 | 不规则数据 |

#### ⚠️ 常见坑

```csharp
// 坑1：第二个括号不能写大小
// int[][] arr = new int[3][4]; // 编译错误！
int[][] arr = new int[3][]; // 正确

// 坑2：忘了给每行 new 就用 → NullReferenceException
int[][] bad = new int[2][];
// bad[0][0] = 1; // 运行时报错：未将对象引用设置到对象的实例
bad[0] = new int[2];  // 必须先 new
bad[0][0] = 1;        // 现在才行
```

---

## 三、值类型和引用类型

### 1. 概念

```csharp
// 值类型：直接存数据本身
// 包括：int、double、bool、char、enum、struct
int a = 10;  // 变量 a 里直接存的就是 10

// 引用类型：存的是数据的地址，数据本身在堆里
// 包括：string、数组、class、interface
int[] arr = { 1, 2, 3 }; // arr 里存的是堆里数组的地址
```

### 2. 赋值差异

```csharp
// 值类型：复制一份，互不影响
int a = 10;
int b = a;
b = 99;
Console.WriteLine(a); // 输出: 10（a 不受影响）
Console.WriteLine(b); // 输出: 99

// 引用类型：共享同一个，改一个两个都变
int[] arr1 = { 1, 2, 3 };
int[] arr2 = arr1;  // arr2 和 arr1 指向同一个数组
arr2[0] = 99;
Console.WriteLine(arr1[0]); // 输出: 99（arr1 也被改了）

// string 特殊：引用类型但不可变，表现得像值类型
string s1 = "hello";
string s2 = s1;
s2 = "world"; // 其实是 s2 指向了新字符串 "world"
Console.WriteLine(s1); // 输出: hello（s1 不受影响）
```

### 3. 方法传参

```csharp
// 值类型传参：传副本，改了不影响外面
void ModifyValue(int x)
{
    x = 100;
}

int num = 10;
ModifyValue(num);
Console.WriteLine(num); // 输出: 10（没变）

// 引用类型传参：传地址副本
void ModifyContent(int[] arr)
{
    arr[0] = 100; // 改内容，影响外面
}

void ModifyRef(int[] arr)
{
    arr = new int[] { 999 }; // 换对象，不影响外面
}

int[] myArr = { 1, 2, 3 };
ModifyContent(myArr);
Console.WriteLine(myArr[0]); // 输出: 100

ModifyRef(myArr);
Console.WriteLine(myArr[0]); // 输出: 100（没换成 999）
```

### 4. 栈和堆

| 对比项 | 栈（Stack） | 堆（Heap） |
|--------|------------|-----------|
| 存什么 | 值类型数据、引用类型的地址 | 引用类型的实际数据 |
| 速度 | 快 | 相对慢 |
| 回收 | 自动，作用域结束就释放 | GC（垃圾回收器）回收 |
| 大小 | 较小 | 较大 |

---

## 四、函数（方法）

### 1. 基本语法

```csharp
// 语法：修饰符 返回类型 函数名(参数) { return 值; }
public static int Add(int a, int b)
{
    return a + b;
}

// void 表示无返回值
public static void SayHi()
{
    Console.WriteLine("Hi!");
    // 没有 return 值，或者直接 return; 提前结束
}

// return：返回结果 + 提前结束
public static int GetScore(int score)
{
    if (score < 0)
    {
        return 0; // 提前结束
    }
    return score;
}

Console.WriteLine(Add(3, 5));    // 输出: 8
SayHi();                          // 输出: Hi!
Console.WriteLine(GetScore(-10)); // 输出: 0
Console.WriteLine(GetScore(88));  // 输出: 88
```

### 2. 局部变量

```csharp
void FuncA()
{
    int x = 10; // 只在 FuncA 内有效
}

void FuncB()
{
    // Console.WriteLine(x); // 编译错误，访问不到
    int x = 20; // 不同函数的局部变量互相独立
}

// 重点：不同函数里同名变量互不干扰
```

### 3. 函数重载

同名函数只要参数列表不同就能共存，编译器根据传入参数自动选合适的。

```csharp
// 参数个数不同
int Add(int a, int b) => a + b;
int Add(int a, int b, int c) => a + b + c;

// 参数类型不同
double Add(double a, double b) => a + b;

// 参数顺序不同
void Show(int a, string b) { }
void Show(string b, int a) { }

Console.WriteLine(Add(1, 2));       // 输出: 3
Console.WriteLine(Add(1, 2, 3));    // 输出: 6
Console.WriteLine(Add(1.5, 2.5));   // 输出: 4
```

#### ⚠️ 重载规则与坑

```csharp
// ❌ 只有返回值不同不算重载
// int Foo(int a);
// void Foo(int a); // 编译错误

// ❌ 只有参数名不同不算重载
// void Foo(int a);
// void Foo(int b); // 编译错误，本质都是 Foo(int)

// ref 和 out 互相不算重载
// void Bar(ref int x);
// void Bar(out int x); // 编译错误，不能这样重载

// 但 ref/out 和普通参数可以重载
void Bar(int x) { }
void Bar(ref int x) { } // 可以
```

#### ⚠️ 坑：重载和默认值产生歧义

```csharp
void Foo(int a, int b = 10) { }
// void Foo(int a) { } // 如果再写这个，调用 Foo(1) 时编译器不知道选哪个
// Foo(1); // 编译错误：调用存在歧义
```

---

## 五、ref 和 out

作用：让函数能修改外面的变量（突破值类型传副本的限制）。

### 1. ref

```csharp
// 经典场景：交换两个变量
void Swap(ref int a, ref int b)
{
    int temp = a;
    a = b;
    b = temp;
}

int x = 1, y = 2;
Swap(ref x, ref y);  // 调用时也要加 ref
Console.WriteLine(x + " " + y); // 输出: 2 1

// 重点：ref 传之前必须先赋值
// int z;
// Swap(ref z, ref y); // 编译错误，z 没赋值
```

### 2. out

```csharp
// 经典场景：TryParse，返回是否成功 + 输出解析结果
bool TryParse(string s, out int result)
{
    if (int.TryParse(s, out result))
        return true;
    result = 0;
    return false;
}

// out 传之前不需要赋值
string input = "123";
if (TryParse(input, out int num))
{
    Console.WriteLine("解析成功: " + num); // 输出: 解析成功: 123
}
```

### 3. ref vs out 对比表

| 对比项 | ref | out |
|--------|-----|-----|
| 传之前是否必须赋值 | ✅ 必须 | ❌ 不需要 |
| 函数内是否必须赋值 | ❌ 可改可不改 | ✅ 必须 |
| 用途 | 修改外部变量 | 输出多个值 |
| 经典场景 | Swap | TryParse |
| 调用时 | 都要加关键字 | 都要加关键字 |
| 能否重载 | ref 和 out 互相不算重载 | 同左 |

#### ⚠️ 常见坑

```csharp
void Foo(ref int x) { }
void Bar(out int x) { x = 0; }

// 坑1：ref 传之前没赋值 → 编译错误
// int a;
// Foo(ref a); // 编译错误

// 坑2：out 函数里没赋值 → 编译错误
// void Bad(out int x) { } // 编译错误，必须给 x 赋值

// 坑3：调用时忘了加 ref/out
int a = 1;
// Foo(a);  // 编译错误，必须写 Foo(ref a)
Foo(ref a); // 正确
```

---

## 六、变长参数和参数默认值

### 1. params

```csharp
// params int[] nums：传几个都行，甚至不传
int Sum(params int[] nums)
{
    int total = 0;
    foreach (int n in nums) total += n;
    return total;
}

Console.WriteLine(Sum());          // 输出: 0
Console.WriteLine(Sum(1));         // 输出: 1
Console.WriteLine(Sum(1, 2, 3));   // 输出: 6
Console.WriteLine(Sum(1, 2, 3, 4, 5)); // 输出: 15

// 规则：必须是数组、必须是最后一个参数、只能有一个 params
```

### 2. 参数默认值

```csharp
// 有默认值的参数，不传就用默认值
void Greet(string name, string greeting = "你好")
{
    Console.WriteLine(greeting + ", " + name);
}

Greet("小明");              // 输出: 你好, 小明
Greet("小红", "Hello");     // 输出: Hello, 小红

// 规则：有默认值的必须排在无默认值的后面
// void Bad(int a = 1, int b) { } // 编译错误

// 默认值必须是常量
// void Bad(string s = GetName()) { } // 编译错误，不是常量

// 命名参数：可以跳过中间的参数
void Info(string name, int age = 18, string city = "北京")
{
    Console.WriteLine($"{name}, {age}岁, {city}");
}

Info("小明", city: "上海"); // 输出: 小明, 18岁, 上海
Info(age: 25, name: "小红"); // 输出: 小红, 25岁, 北京
```

### 3. 参数顺序：普通参数 → 默认参数 → params

```csharp
void Process(string name, int age = 18, params string[] hobbies)
{
    Console.Write($"{name}, {age}岁");
    foreach (string h in hobbies)
        Console.Write(", " + h);
    Console.WriteLine();
}

Process("小明");                        // 输出: 小明, 18岁
Process("小红", 20);                    // 输出: 小红, 20岁
Process("小刚", 22, "读书", "打球");     // 输出: 小刚, 22岁, 读书, 打球
```

---

## 七、递归

函数自己调用自己。

### 两个必要条件
1. **终止条件**：什么时候停下来
2. **缩小规模**：每次调用问题要变小

### 示例：阶乘

```csharp
// n! = n × (n-1)!，1! = 1
int Factorial(int n)
{
    if (n <= 1) return 1;     // 终止条件
    return n * Factorial(n - 1); // 缩小规模
}

Console.WriteLine(Factorial(5)); // 输出: 120
// 过程：5 * 4 * 3 * 2 * 1 = 120
```

### 示例：斐波那契

```csharp
// F(n) = F(n-1) + F(n-2)，F(1)=F(2)=1
int Fib(int n)
{
    if (n <= 2) return 1;      // 终止条件
    return Fib(n - 1) + Fib(n - 2); // 缩小规模
}

Console.WriteLine(Fib(6)); // 输出: 8
// 序列：1 1 2 3 5 8
```

### 递归 vs 循环对比

| 对比项 | 递归 | 循环 |
|--------|------|------|
| 可读性 | 好（接近数学定义） | 一般 |
| 性能 | 差（函数调用开销） | 好 |
| 风险 | 太深会栈溢出 | 安全 |
| 使用场景 | 树形结构、分治 | 大多数场景 |

#### ⚠️ 常见坑

```csharp
// 坑1：忘终止条件 → StackOverflowException
// int Bad(int n) { return Bad(n - 1); } // 死递归，栈溢出

// 坑2：不缩小规模 → 死循环
// int Bad2(int n)
// {
//     if (n <= 0) return 0;
//     return Bad2(n); // 没缩小！永远停不下来
// }
```

---

## 八、结构体（struct）

把相关字段打包在一起，类似轻量级的类。

```csharp
// 定义结构体
struct Point
{
    // 字段
    public int X;
    public int Y;

    // 构造函数：必须给所有字段赋值
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    // 可以有方法
    public void Show()
    {
        Console.WriteLine($"({X}, {Y})");
    }
}

Point p1 = new Point(3, 4);
p1.Show(); // 输出: (3, 4)

// 结构体是值类型！赋值是复制，不是共享
Point p2 = p1;  // 复制一份
p2.X = 99;
Console.WriteLine(p1.X); // 输出: 3（p1 不受影响）
Console.WriteLine(p2.X); // 输出: 99
```

### struct vs class 对比表

| 对比项 | struct（结构体） | class（类） |
|--------|-----------------|------------|
| 类型 | 值类型 | 引用类型 |
| 赋值 | 复制（互不影响） | 共享地址（改一个都变） |
| 存储 | 栈 | 堆 |
| 继承 | ❌ 不能继承 | ✅ 可以继承 |
| 构造函数 | 必须给所有字段赋值 | 自由 |
| 使用场景 | 轻量数据（坐标、颜色） | 复杂对象 |

#### ⚠️ 常见坑

```csharp
struct Student
{
    public int Age;
    public string Name;

    // 坑1：构造函数没给所有字段赋值 → 编译错误
    // public Student(int age)
    // {
    //     Age = age;
    //     // 编译错误：Name 没赋值
    // }

    public Student(int age, string name)
    {
        Age = age;
        Name = name; // 所有字段都要赋值
    }
}

// 坑2：以为赋值是共享（值类型是复制）
Student s1 = new Student(18, "小明");
Student s2 = s1; // 这是复制！
s2.Name = "小红";
Console.WriteLine(s1.Name); // 输出: 小明（没变）
Console.WriteLine(s2.Name); // 输出: 小红
```

---

## 九、排序

### 1. 冒泡排序

**思想**：相邻两两比较，大的往后挪，每轮把最大的"冒泡"到末尾。

```csharp
int[] arr = { 5, 3, 8, 1, 9 };

// 外层 n-1 轮，内层 n-1-i 次
for (int i = 0; i < arr.Length - 1; i++)
{
    bool swapped = false; // 优化标记
    for (int j = 0; j < arr.Length - 1 - i; j++)
    {
        // 升序用 >，降序用 <
        if (arr[j] > arr[j + 1])
        {
            int temp = arr[j];
            arr[j] = arr[j + 1];
            arr[j + 1] = temp;
            swapped = true;
        }
    }
    // 优化：一轮没交换说明已经有序，提前结束
    if (!swapped) break;
}

// 输出结果
foreach (int x in arr) Console.Write(x + " ");
// 输出: 1 3 5 8 9
```

#### ⚠️ 常见坑

```csharp
// 坑1：交换忘用 temp
// arr[j] = arr[j + 1];
// arr[j + 1] = arr[j]; // 错！这样 arr[j+1] 已经被覆盖，丢失原值

// 坑2：内层忘减 i
// for (int j = 0; j < arr.Length - 1; j++) // 错！没减 i 会重复比较已排好的部分
// 正确：for (int j = 0; j < arr.Length - 1 - i; j++)
```

### 2. 选择排序

**思想**：每轮找最小值的下标，放到前面，每轮只交换一次。

```csharp
int[] arr = { 5, 3, 8, 1, 9 };

for (int i = 0; i < arr.Length - 1; i++)
{
    int minIndex = i; // 记录最小值的下标
    for (int j = i + 1; j < arr.Length; j++)
    {
        // 升序找最小（<），降序找最大（>）
        if (arr[j] < arr[minIndex])
        {
            minIndex = j;
        }
    }
    // 每轮只交换一次
    if (minIndex != i)
    {
        int temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
}

foreach (int x in arr) Console.Write(x + " ");
// 输出: 1 3 5 8 9
```

### 3. 冒泡 vs 选择 对比表

| 对比项 | 冒泡排序 | 选择排序 |
|--------|---------|---------|
| 比较次数 | n(n-1)/2 | n(n-1)/2（一样） |
| 交换次数 | 多（每轮可能多次） | 少（每轮最多1次） |
| 稳定性 | 稳定（相等元素不交换） | 不稳定（可能改变相等元素顺序） |
| 最好情况 | O(n)（已有序，加优化） | O(n²)（仍要全部比较） |
| 实现难度 | 简单 | 简单 |

### 4. 内置排序

实际开发别手写排序，用内置的就行：

```csharp
int[] arr = { 5, 3, 8, 1, 9 };

Array.Sort(arr); // 升序排序
foreach (int x in arr) Console.Write(x + " ");
// 输出: 1 3 5 8 9

// 降序：先升序再反转
Array.Sort(arr);
Array.Reverse(arr);
foreach (int x in arr) Console.Write(x + " ");
// 输出: 9 8 5 3 1
```

---

## 进阶篇总结

### 每个知识点一句话总结

| 知识点 | 一句话总结 |
|--------|-----------|
| 枚举 | 给一组数值起名字，代码更可读，配 switch 是黄金搭档 |
| 数组 | 固定长度的同类型集合，二维用 `[,]`，交错用 `[][]` |
| 值类型 | 存数据本身，赋值是复制；引用类型存地址，赋值是共享 |
| 函数 | 封装一段逻辑，重载让同名函数各司其职 |
| ref/out | 让函数能修改外部变量，ref 必须先赋值，out 必须在函数内赋值 |
| params/默认值 | params 让参数数量灵活，默认值让参数可省略 |
| 递归 | 自己调用自己，必须有终止条件 + 缩小规模 |
| 结构体 | 值类型的轻量数据封装，赋值是复制不是共享 |
| 排序 | 冒泡两两比较，选择找极值，实际开发用 Array.Sort |

### 学习建议

1. **枚举和 switch 一起练**：写一个根据状态码返回描述的小程序
2. **数组的坑要踩一遍**：亲手试一下引用赋值不是复制，加深印象
3. **值类型 vs 引用类型是核心**：搞懂这个，后面学 class 事半功倍
4. **ref/out 多写例子**：Swap 和 TryParse 自己实现一遍
5. **排序手写一遍就够**：理解思想，实际开发用内置的
6. **结构体和类对比着学**：先理解 struct 是值类型，再学 class 会更轻松
7. **多写小例子**：每个知识点写个 10 行的小程序，比看十遍教程都管用

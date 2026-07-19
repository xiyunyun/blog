---
title: C#学习【入门篇】
published: 2026-07-17
pinned: false
description: "此文档记录了C#入门的学习内容"
image: "./Csharp_Image/Beginner.avif"
tags: ["编程","C#"]
category: 学习记录
slug: Csharp_P1
---

# C# 入门篇

> 本篇覆盖从变量到循环的基础语法，学完能写简单的控制台程序。

---

## 一、变量和数据类型

### 1. 值类型与引用类型

- **值类型**：`int`、`float`、`double`、`bool`、`char`、`decimal`、`long` 等，存储在栈上，赋值是复制值。
- **引用类型**：`string`、`object`、数组、类等，存储在堆上，赋值是复制引用。
- **注意**：`string` 虽然是引用类型，但用法像值类型（不可变，比较用 `==` 比内容）。

### 2. 变量声明

```csharp
// 语法：类型 变量名 = 值;
int age = 18;
float score = 95.5f;     // 注意 f 后缀
double pi = 3.14159;     // double 不用加后缀
bool isStudent = true;
char grade = 'A';        // char 用单引号
string name = "小明";    // string 用双引号
decimal money = 99.99m;  // 注意 m 后缀

Console.WriteLine($"{name} 今年 {age} 岁，成绩 {score}"); 
// 输出：小明 今年 18 岁，成绩 95.5
```

### 3. 变量命名规则

- 只能由**字母、数字、下划线**组成。
- **不能以数字开头**。
- **不能使用关键字**（如 `int`、`class`、`if`）。
- 区分大小写：`age` 和 `Age` 是不同的变量。

```csharp
int _count = 0;       // 合法
int age2 = 18;         // 合法
// int 2age = 18;     // 错误：数字开头
// int int = 5;       // 错误：用了关键字
string userName = "Tom";  // 驼峰命名推荐
```

### 4. 常量

```csharp
const int Max = 100;
// Max = 200;  // 错误：常量不能重新赋值
Console.WriteLine(Max);  // 输出：100
```

### 5. var 隐式推断

```csharp
var x = 10;        // 编译时推断为 int
var s = "hello";   // 推断为 string
var d = 3.14;      // 推断为 double
// var y;          // 错误：var 必须初始化才能推断类型
```

> `var` 只是语法糖，编译后类型已确定，**不是动态类型**。

### 6. 基本类型及范围对比表

| 类型       | 说明         | 范围 / 取值                                  | 后缀 | 默认值   |
| ---------- | ------------ | -------------------------------------------- | ---- | -------- |
| `sbyte`    | 8 位有符号   | -128 ~ 127                                   | -    | 0        |
| `byte`     | 8 位无符号   | 0 ~ 255                                      | -    | 0        |
| `short`    | 16 位有符号  | -32,768 ~ 32,767                             | -    | 0        |
| `ushort`   | 16 位无符号  | 0 ~ 65,535                                   | -    | 0        |
| `int`      | 32 位有符号  | -2,147,483,648 ~ 2,147,483,647               | -    | 0        |
| `uint`     | 32 位无符号  | 0 ~ 4,294,967,295                            | `u`  | 0        |
| `long`     | 64 位有符号  | -9.2×10^18 ~ 9.2×10^18                       | `L`  | 0        |
| `ulong`    | 64 位无符号  | 0 ~ 1.8×10^19                                | `ul` | 0        |
| `float`    | 单精度浮点   | ±1.5×10^-45 ~ ±3.4×10^38（约 7 位有效数字）  | `f`  | 0        |
| `double`   | 双精度浮点   | ±5.0×10^-324 ~ ±1.7×10^308（约 15 位有效数字）| -    | 0        |
| `decimal`  | 高精度小数   | ±1.0×10^-28 ~ ±7.9×10^28（28 位有效数字）    | `m`  | 0        |
| `bool`     | 布尔         | true / false                                 | -    | false    |
| `char`     | 单个字符     | U+0000 ~ U+FFFF                              | -    | '\0'     |
| `string`   | 字符串       | 任意长度文本                                 | -    | null     |

### ⚠️ 常见坑

1. **`float` 赋值必须加 `f` 后缀**，否则编译器默认当 `double`，会报错：
   ```csharp
   float f = 3.14f;   // 正确
   // float f2 = 3.14;  // 错误：默认是 double，无法隐式转 float
   ```
2. **`decimal` 赋值必须加 `m` 后缀**（money 的 m，常用于货币计算）：
   ```csharp
   decimal price = 9.99m;  // 正确
   // decimal p2 = 9.99;    // 错误
   ```
3. **`double` 不用加后缀**，它是浮点数默认类型。
4. **变量必须先声明再使用**，且作用域仅在声明它的 `{}` 内：
   ```csharp
   if (true)
   {
       int x = 5;
   }
   // Console.WriteLine(x);  // 错误：x 已超出作用域
   ```

---

## 二、运算符

### 1. 算术运算符

| 运算符 | 说明   | 示例            | 结果 |
| ------ | ------ | --------------- | ---- |
| `+`    | 加     | `5 + 3`         | 8    |
| `-`    | 减     | `5 - 3`         | 2    |
| `*`    | 乘     | `5 * 3`         | 15   |
| `/`    | 除     | `5 / 2`         | 2    |
| `%`    | 取余   | `5 % 2`         | 1    |

```csharp
Console.WriteLine(5 / 2);     // 输出：2（整数除法，不是 2.5）
Console.WriteLine(5.0 / 2);  // 输出：2.5（有浮点数参与就是浮点除法）
Console.WriteLine(10 % 3);   // 输出：1
Console.WriteLine(-7 % 3);   // 输出：-1（结果符号跟被除数）
```

> **坑**：整数除以整数，结果还是整数（直接截断小数部分）。要得到小数结果，必须让至少一个操作数是浮点数。

#### 自增自减：`++` `--`

```csharp
int a = 5;
Console.WriteLine(a++);  // 输出：5（先用再 +1，后缀）
Console.WriteLine(a);    // 输出：6

int b = 5;
Console.WriteLine(++b);  // 输出：6（先 +1 再用，前缀）
Console.WriteLine(b);    // 输出：6
```

> **记忆**：`++` 在前先加再用，`++` 在后先用再加。

### 2. 赋值运算符

```csharp
int n = 10;
n += 5;   // 等价 n = n + 5;  → 15
n -= 3;   // 等价 n = n - 3;  → 12
n *= 2;   // 等价 n = n * 2;  → 24
n /= 4;   // 等价 n = n / 4;  → 6
n %= 4;   // 等价 n = n % 4;  → 2
Console.WriteLine(n);  // 输出：2
```

### 3. 关系运算符

```csharp
int a = 5, b = 10;
Console.WriteLine(a == b);  // 输出：False
Console.WriteLine(a != b);  // 输出：True
Console.WriteLine(a > b);   // 输出：False
Console.WriteLine(a < b);   // 输出：True
Console.WriteLine(a >= 5);  // 输出：True
Console.WriteLine(a <= 4);  // 输出：False
```

### 4. 逻辑运算符

```csharp
bool a = true, b = false;
Console.WriteLine(a && b);  // 输出：False（与：都真才真）
Console.WriteLine(a || b);  // 输出：True（或：有真即真）
Console.WriteLine(!a);      // 输出：False（非：取反）
```

#### 短路特性

- `&&`：左边为 `false` 时，**右边不再计算**。
- `||`：左边为 `true` 时，**右边不再计算**。

```csharp
int x = 0;
// x == 0 为 true，右边 10 / x > 1 不计算，避免除零异常
if (x != 0 && 10 / x > 1)
{
    Console.WriteLine("不会执行");
}
Console.WriteLine("短路保护成功");  // 输出：短路保护成功
```

### 5. 位运算符

```csharp
int a = 5;    // 0101
int b = 3;    // 0011
Console.WriteLine(a & b);   // 输出：1  (0001，按位与)
Console.WriteLine(a | b);   // 输出：7  (0111，按位或)
Console.WriteLine(a ^ b);   // 输出：6  (0110，按位异或)
Console.WriteLine(~a);      // 输出：-6 (按位取反)
Console.WriteLine(a << 1);  // 输出：10 (左移 1 位相当于 ×2)
Console.WriteLine(a >> 1);   // 输出：2  (右移 1 位相当于 ÷2 取整)
```

### 6. 条件运算符（三元）

```csharp
int age = 20;
string result = age >= 18 ? "成年" : "未成年";
Console.WriteLine(result);  // 输出：成年
```

### 7. 运算符优先级

**口诀**：括号 > 单目 > 算术 > 关系 > 逻辑 > 条件 > 赋值

| 优先级（高→低） | 运算符                                   |
| --------------- | ---------------------------------------- |
| 1               | `()` 括号                                |
| 2               | `!` `~` `++` `--`（单目）                 |
| 3               | `*` `/` `%`                              |
| 4               | `+` `-`                                  |
| 5               | `<<` `>>`                                |
| 6               | `<` `>` `<=` `>=`                        |
| 7               | `==` `!=`                                |
| 8               | `&`                                      |
| 9               | `^`                                      |
| 10              | `|`                                      |
| 11              | `&&`                                     |
| 12              | `||`                                     |
| 13              | `?:`（三元）                             |
| 14              | `=` `+=` `-=` 等（赋值）                  |

> **建议**：拿不准就加括号，可读性最重要。

---

## 三、类型转换

### 1. 隐式转换（小转大，安全）

```csharp
int i = 100;
long l = i;         // int → long，安全
float f = l;        // long → float，自动转换
double d = f;       // float → double，自动转换
Console.WriteLine(d);  // 输出：100
```

转换链：`int → long → float → double`

> **坑**：`long → float` 会丢精度，因为 `float` 只有约 7 位有效数字。
> ```csharp
> long big = 123456789L;
> float f2 = big;
> Console.WriteLine(f2);  // 输出：1.2345679E+08（精度丢失！）
> ```

> **坑**：运算中 `byte + byte` 会自动提升为 `int`。
> ```csharp
> byte b1 = 1, b2 = 2;
> // byte b3 = b1 + b2;   // 错误：结果是 int，需强转
> byte b3 = (byte)(b1 + b2);  // 正确
> int b4 = b1 + b2;          // 正确
> ```

### 2. 显式转换（大转小，可能丢数据）

```csharp
double d = 3.9;
int i = (int)d;
Console.WriteLine(i);  // 输出：3（直接截断小数部分，不是四舍五入）
```

#### 字符串转数字

```csharp
// int.Parse：转换失败抛异常
int n1 = int.Parse("123");
Console.WriteLine(n1);  // 输出：123
// int.Parse("abc");    // 抛 FormatException

// int.TryParse：推荐！失败返回 false，不抛异常
if (int.TryParse("123", out int n2))
{
    Console.WriteLine(n2);  // 输出：123
}

if (!int.TryParse("abc", out int n3))
{
    Console.WriteLine("转换失败");  // 输出：转换失败
}

// Convert.ToInt32：银行家舍入（四舍六入五取偶）
int n4 = Convert.ToInt32(3.9);
Console.WriteLine(n4);  // 输出：4

int n5 = Convert.ToInt32(2.5);
Console.WriteLine(n5);  // 输出：2（5 取偶，向偶数舍入）

int n6 = Convert.ToInt32(3.5);
Console.WriteLine(n6);  // 输出：4（5 取偶，向偶数舍入）
```

### 3. Convert vs Parse vs 强转 对比表

| 方式                | 语法                    | 转换对象     | 失败行为           | 舍入方式           | 推荐场景             |
| ------------------- | ----------------------- | ------------ | ------------------ | ------------------ | -------------------- |
| 强转 `(int)`        | `(int)3.9`              | 数值→数值    | 无（直接截断）     | 截断（向零取整）   | 浮点转整数           |
| `Convert.ToInt32`   | `Convert.ToInt32(x)`    | 多种类型     | 抛 `FormatException` | 银行家舍入（四舍六入五取偶）| 字符串/对象转整数 |
| `int.Parse`         | `int.Parse("123")`      | 仅字符串     | 抛 `FormatException` | -                  | 确定字符串一定是数字 |
| `int.TryParse`      | `int.TryParse(s, out n)`| 仅字符串     | 返回 `false`，不抛异常 | -                | 用户输入等不可信数据 |

---

## 四、字符串

### 1. 字符串拼接

```csharp
string name = "小明";
int age = 18;

// 方式一：+ 拼接（简单，少量时用）
string s1 = "姓名：" + name + "，年龄：" + age;
Console.WriteLine(s1);  // 输出：姓名：小明，年龄：18

// 方式二：$ 插值（推荐！最直观）
string s2 = $"姓名：{name}，年龄：{age}";
Console.WriteLine(s2);  // 输出：姓名：小明，年龄：18

// 方式三：string.Format
string s3 = string.Format("姓名：{0}，年龄：{1}", name, age);
Console.WriteLine(s3);  // 输出：姓名：小明，年龄：18

// 方式四：StringBuilder（大量拼接时用，性能好）
var sb = new System.Text.StringBuilder();
for (int i = 0; i < 3; i++)
{
    sb.Append($"第{i + 1}行\n");
}
Console.WriteLine(sb.ToString());
// 输出：
// 第1行
// 第2行
// 第3行
```

### 2. 转义字符

| 转义符 | 含义     | 示例                |
| ------ | -------- | ------------------- |
| `\n`   | 换行     | `"a\nb"` → a/b 两行 |
| `\t`   | 制表符   | `"a\tb"` → a    b   |
| `\"`   | 双引号   | `"说\"你好\""`      |
| `\\`   | 反斜杠   | `"C:\\Users"`       |
| `\r`   | 回车     | `"a\rb"`            |
| `\0`   | 空字符   | -                   |

#### `@` 逐字字符串

```csharp
string path = @"C:\Users\name\file.txt";  // 不用写双反斜杠
Console.WriteLine(path);  // 输出：C:\Users\name\file.txt

string multi = @"第一行
第二行
第三行";  // 逐字字符串可跨行
Console.WriteLine(multi);
// 输出：
// 第一行
// 第二行
// 第三行
```

### 3. 常用方法

```csharp
string s = "Hello World";

Console.WriteLine(s.Length);           // 输出：11（长度）
Console.WriteLine(s.ToUpper());        // 输出：HELLO WORLD
Console.WriteLine(s.ToLower());        // 输出：hello world
Console.WriteLine(s.Substring(0, 5));  // 输出：Hello（从索引 0 取 5 个字符）
Console.WriteLine(s.IndexOf("World")); // 输出：6（首次出现的索引）
Console.WriteLine(s.IndexOf("xyz"));   // 输出：-1（未找到）
Console.WriteLine(s.Replace("World", "C#"));  // 输出：Hello C#
Console.WriteLine(s.Contains("Hello"));      // 输出：True
Console.WriteLine("  abc  ".Trim());          // 输出：abc（去首尾空格）

string[] parts = "a,b,c".Split(',');
foreach (string p in parts)
{
    Console.WriteLine(p);
}
// 输出：
// a
// b
// c
```

### 4. 字符串不可变性

C# 中 `string` 是**不可变**的：每次修改都会创建一个新字符串对象，原字符串不变。

```csharp
string s = "a";
s += "b";  // 不是修改 s，而是创建新字符串 "ab"，s 指向新对象
Console.WriteLine(s);  // 输出：ab
```

> **坑**：在循环中大量用 `+` 拼接字符串，会产生大量临时对象，性能差。
> ```csharp
> // 反例：每次 += 都创建新字符串
> string bad = "";
> for (int i = 0; i < 1000; i++)
> {
>     bad += i.ToString();  // 性能差
> }
>
> // 正例：用 StringBuilder
> var good = new System.Text.StringBuilder();
> for (int i = 0; i < 1000; i++)
> {
>     good.Append(i);  // 性能好
> }
> ```

---

## 五、控制流 - 条件判断

### 1. if 语句

```csharp
int score = 85;

// if
if (score >= 60)
{
    Console.WriteLine("及格");  // 输出：及格
}

// if-else
if (score >= 60)
{
    Console.WriteLine("及格");
}
else
{
    Console.WriteLine("不及格");
}

// if-else if-else
if (score >= 90)
{
    Console.WriteLine("优秀");
}
else if (score >= 80)
{
    Console.WriteLine("良好");  // 输出：良好
}
else if (score >= 60)
{
    Console.WriteLine("及格");
}
else
{
    Console.WriteLine("不及格");
}
```

> **坑**：条件必须是 `bool` 类型，不能写成 `if (5)`，这和 C/C++ 不同。
> ```csharp
> // if (5) { }  // 编译错误：int 不能隐式转 bool
> if (5 > 3) { }  // 正确
> ```

> **坑**：`= `和 `==` 写混。`if (x = 5)` 在 C# 中是**编译错误**（因为 `int` 不能转 `bool`），这其实是 C# 的保护机制。
> ```csharp
> int x = 0;
> // if (x = 5) { }  // 编译错误：无法将 int 转为 bool
> if (x == 5) { }   // 正确
> ```

### 2. switch 语句

#### 基本语法

```csharp
int day = 3;
switch (day)
{
    case 1:
        Console.WriteLine("星期一");
        break;
    case 2:
        Console.WriteLine("星期二");
        break;
    case 3:
        Console.WriteLine("星期三");  // 输出：星期三
        break;
    default:
        Console.WriteLine("未知");
        break;
}
```

#### case 合并（空 case 共享代码）

```csharp
int month = 7;
switch (month)
{
    case 3:
    case 4:
    case 5:
        Console.WriteLine("春季");
        break;
    case 6:
    case 7:
    case 8:
        Console.WriteLine("夏季");  // 输出：夏季
        break;
    default:
        Console.WriteLine("其他季节");
        break;
}
```

#### 支持的类型

`switch` 支持 `int`、`char`、`string`、`enum` 等：

```csharp
string cmd = "start";
switch (cmd)
{
    case "start":
        Console.WriteLine("开始");  // 输出：开始
        break;
    case "stop":
        Console.WriteLine("停止");
        break;
    default:
        Console.WriteLine("未知命令");
        break;
}
```

#### C# 7.0+ 模式匹配（简述）

```csharp
object obj = "hello";
switch (obj)
{
    case int n when n > 0:
        Console.WriteLine($"正整数：{n}");
        break;
    case string s:
        Console.WriteLine($"字符串：{s}");  // 输出：字符串：hello
        break;
    case null:
        Console.WriteLine("空值");
        break;
    default:
        Console.WriteLine("其他类型");
        break;
}
```

#### switch 表达式（C# 8.0+，简述）

```csharp
int day = 3;
string dayName = day switch
{
    1 => "星期一",
    2 => "星期二",
    3 => "星期三",  // 输出：星期三
    _ => "其他"
};
Console.WriteLine(dayName);
```

> **坑**：C# 中 `case` 后**必须**有 `break`（或 `return` / `goto` / `throw`），**不能像 C/C++ 那样穿透**！
> ```csharp
> // int x = 1;
> // switch (x)
> // {
> //     case 1:
> //         Console.WriteLine("一");
> //     case 2:                  // 错误：控制不能从一个 case 穿透到另一个
> //         Console.WriteLine("二");
> //         break;
> // }
> ```

---

## 六、控制流 - 循环

### 1. while 循环

```csharp
int i = 1;
while (i <= 3)
{
    Console.WriteLine($"第 {i} 次");
    i++;
}
// 输出：
// 第 1 次
// 第 2 次
// 第 3 次
```

特点：**先判断再执行**，条件不满足时一次都不执行。

> **坑**：忘改条件变量导致死循环。
> ```csharp
> // int i = 1;
> // while (i <= 3)
> // {
> //     Console.WriteLine(i);  // 死循环！i 永远是 1
> // }
> ```

### 2. do-while 循环

```csharp
int i = 1;
do
{
    Console.WriteLine($"第 {i} 次");
    i++;
} while (i <= 3);
// 输出：
// 第 1 次
// 第 2 次
// 第 3 次
```

特点：**先执行再判断**，至少执行一次。

```csharp
// 即使条件一开始就为假，也执行一次
int n = 10;
do
{
    Console.WriteLine(n);  // 输出：10
} while (n < 5);
```

> **坑**：`while(条件)` 后面**必须有分号**！
> ```csharp
> // do { } while (true)   // 错误：少分号
> do { } while (true);    // 正确
> ```

### 3. for 循环

```csharp
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine(i);
}
// 输出：
// 1
// 2
// 3
// 4
// 5
```

三要素：`初始化; 条件; 迭代`

```csharp
// 倒序
for (int i = 5; i >= 1; i--)
{
    Console.WriteLine(i);  // 输出：5 4 3 2 1
}

// 累加 1 到 100
int sum = 0;
for (int i = 1; i <= 100; i++)
{
    sum += i;
}
Console.WriteLine(sum);  // 输出：5050
```

### 4. foreach 循环

```csharp
string[] names = { "张三", "李四", "王五" };
foreach (string name in names)
{
    Console.WriteLine(name);
}
// 输出：
// 张三
// 李四
// 王五
```

语法：`foreach (类型 变量 in 集合)`

> **坑**：`foreach` 是**只读遍历**，不能修改集合元素。
> ```csharp
> int[] nums = { 1, 2, 3 };
> // foreach (int n in nums)
> // {
> //     n = 0;  // 错误：不能给 foreach 迭代变量赋值
> // }
> ```

### 5. break 和 continue

```csharp
// break：跳出整个循环
for (int i = 1; i <= 10; i++)
{
    if (i == 5) break;
    Console.WriteLine(i);
}
// 输出：1 2 3 4

// continue：跳过本次，继续下一次
for (int i = 1; i <= 5; i++)
{
    if (i == 3) continue;
    Console.WriteLine(i);
}
// 输出：1 2 4 5
```

### 6. 四种循环对比表

| 循环类型     | 执行顺序       | 最少执行次数 | 适用场景                   |
| ------------ | -------------- | ------------ | -------------------------- |
| `while`      | 先判断后执行   | 0 次         | 不确定次数，满足条件才执行 |
| `do-while`   | 先执行后判断   | 1 次         | 至少要执行一次             |
| `for`        | 先判断后执行   | 0 次         | 已知循环次数               |
| `foreach`    | 顺序遍历       | 0 次         | 遍历集合/数组（只读）      |

### 7. 嵌套循环 - 九九乘法表

```csharp
for (int i = 1; i <= 9; i++)
{
    for (int j = 1; j <= i; j++)
    {
        Console.Write($"{j}×{i}={i * j}\t");
    }
    Console.WriteLine();
}
// 输出：
// 1×1=1
// 1×2=2    2×2=4
// 1×3=3    2×3=6    3×3=9
// ...（依此类推到 9×9=81）
```

---

## 七、异常捕获

### 1. try-catch

```csharp
try
{
    int n = int.Parse("abc");  // 会抛 FormatException
    Console.WriteLine(n);
}
catch (FormatException ex)
{
    Console.WriteLine("格式错误：" + ex.Message);  // 输出：格式错误：...
}
```

#### 多个 catch（从具体到通用）

```csharp
try
{
    int[] arr = { 1, 2, 3 };
    Console.WriteLine(arr[5]);  // 越界
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine("数组越界");  // 输出：数组越界
}
catch (Exception ex)  // 兜底，必须放最后
{
    Console.WriteLine("其他异常：" + ex.Message);
}
```

> **坑**：多个 `catch` 块时，**具体异常在前，通用异常（`Exception`）在最后**，否则编译错误。

### 2. try-catch-finally

```csharp
try
{
    int a = 10, b = 0;
    int c = a / b;  // 抛 DivideByZeroException
    Console.WriteLine(c);
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("除零错误：" + ex.Message);  // 输出：除零错误：...
}
finally
{
    Console.WriteLine("无论如何都执行");  // 输出：无论如何都执行
}
```

`finally` 无论是否发生异常都会执行，常用于**资源释放**。

### 3. 异常对象

```csharp
try
{
    string s = null;
    Console.WriteLine(s.Length);  // 抛 NullReferenceException
}
catch (NullReferenceException ex)
{
    Console.WriteLine("错误信息：" + ex.Message);       // 简短描述
    Console.WriteLine("堆栈跟踪：\n" + ex.StackTrace);  // 调用堆栈
}
```

常用属性：
- `ex.Message`：错误简短描述。
- `ex.StackTrace`：调用堆栈，定位异常位置。
- `ex.GetType().Name`：异常类型名。

### 4. throw - 主动抛异常

```csharp
public static int Divide(int a, int b)
{
    if (b == 0)
    {
        throw new DivideByZeroException("除数不能为零");  // 主动抛出
    }
    return a / b;
}

// 调用
try
{
    Console.WriteLine(Divide(10, 0));
}
catch (DivideByZeroException ex)
{
    Console.WriteLine(ex.Message);  // 输出：除数不能为零
}
```

`throw` 还可以重新抛出当前异常（保留原始堆栈）：
```csharp
try { /* ... */ }
catch (Exception ex)
{
    // 记录日志
    throw;  // 重新抛出，保留原始堆栈（不要写 throw ex，会重置堆栈）
}
```

### 5. 常见异常类型表

| 异常类型                  | 触发场景                       | 示例                              |
| ------------------------- | ------------------------------ | --------------------------------- |
| `FormatException`         | 字符串格式不正确               | `int.Parse("abc")`                |
| `DivideByZeroException`   | 整数除以零                     | `int x = 5 / 0;`                  |
| `IndexOutOfRangeException` | 数组索引越界                  | `arr[arr.Length]`                 |
| `NullReferenceException`  | 访问 null 对象成员             | `null.ToString()`                 |
| `OverflowException`       | 算术运算溢出（checked 中）     | `checked { int x = int.MaxValue + 1; }` |
| `InvalidCastException`    | 无效的类型转换                 | `(int)(object)"abc"`              |
| `ArgumentNullException`   | 方法参数为 null（不合法）      | 内部主动抛出                      |
| `ArgumentOutOfRangeException` | 参数超出有效范围           | `s.Substring(-1, 5)`              |
| `StackOverflowException`  | 栈溢出（如无限递归）           | 无限递归调用                      |
| `OutOfMemoryException`    | 内存不足                       | 分配超大数组                      |

### 6. 最佳实践

1. **不要空 catch**（吞掉异常会让 bug 难以排查）：
   ```csharp
   // 反例：吞掉异常
   try { DoSomething(); }
   catch { }  // 错误示范：什么都没做
   
   // 正例：至少记录日志
   try { DoSomething(); }
   catch (Exception ex)
   {
       Console.WriteLine($"出错：{ex.Message}");
   }
   ```

2. **能预判就提前判断，别用异常做流程控制**：
   ```csharp
   // 反例：用异常判断输入
   // try { int n = int.Parse(input); }
   // catch { Console.WriteLine("输入非法"); }
   
   // 正例：先判断
   if (int.TryParse(input, out int n))
   {
       Console.WriteLine(n);
   }
   else
   {
       Console.WriteLine("输入非法");
   }
   ```

3. **catch 具体类型**，不要一上来就 `catch (Exception)`。

4. **finally 用于资源释放**，确保即使异常也能关闭文件、数据库连接等。

---

## 入门篇总结

### 一句话总结每个知识点

| 知识点       | 一句话总结                                                     |
| ------------ | -------------------------------------------------------------- |
| 变量与类型   | 值类型存值、引用类型存引用；`float` 加 `f`、`decimal` 加 `m`   |
| 运算符       | 整数除法得整数；`&&` `||` 有短路；拿不准加括号                 |
| 类型转换     | 小转大自动，大转小强转；字符串转数字优先 `TryParse`            |
| 字符串       | `string` 不可变；拼接用 `$`，大量拼接用 `StringBuilder`        |
| 条件判断     | `if` 条件必须是 `bool`；`switch` 的 `case` 必须有 `break`      |
| 循环         | `while` 先判后执，`do-while` 先执后判；`foreach` 只读          |
| 异常         | `try-catch-finally`；别空 catch，别用异常做流程控制           |

### 学习建议

1. **多写代码**：看懂不等于会写，每个语法点都自己敲一遍。
2. **善用断点调试**：F9 设断点，F10 单步执行，F11 进入方法，观察变量变化。
3. **看报错信息**：编译错误和异常信息是 Debug 的第一线索，别一报错就慌。
4. **从简单程序开始**：先写计算器、猜数字、九九乘法表等小项目，巩固基础。
5. **官方文档是好帮手**：[Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/) 有最权威的教程和示例。

> 学完入门篇，你已经能写出简单的控制台程序了。下一篇进阶篇将带你进入方法、数组、面向对象的世界！

---
title: C#学习【高级篇】
published: 2026-07-17
pinned: false
description: "此文档记录了C#高级的学习内容"
image: "./Csharp_Image/Advanced.avif"
tags: ["编程","C#"]
category: 学习记录
slug: Csharp_P4
---

# C# 高级篇

> 本篇覆盖集合、泛型、委托事件、多线程、反射特性和高级排序，学完能写出专业级 C# 程序。

---

## 一、集合

### 1. List<T>（动态数组）

List<T> 就是一个会自动变长的数组，最常用的集合。

**声明与常用方法：**

```csharp
List<int> list = new List<int>();
list.Add(1);                    // 添加单个元素
list.AddRange(new[] { 2, 3 });  // 添加多个元素
list.Remove(2);                 // 删除元素 2
list.RemoveAt(0);               // 删除索引 0 处的元素
bool has = list.Contains(3);    // 是否包含 3 → True
int idx = list.IndexOf(3);      // 查索引，找不到返回 -1
list.Sort();                    // 排序
list.Clear();                   // 清空
// 此时 list.Count → 0
```

**Count vs Capacity：**

```csharp
List<int> list = new List<int>();
for (int i = 0; i < 5; i++) list.Add(i);
Console.WriteLine(list.Count);      // 5（实际元素数）
Console.WriteLine(list.Capacity);   // 8（底层分配的容量，会按倍数扩张）
```

**坑：遍历时删除元素会跳过**（因为删除后后面元素前移，索引却继续 +1）：

```csharp
List<int> nums = new List<int> { 1, 2, 2, 3, 4 };
// 错误写法：会漏删
foreach (var n in nums) { /* 不能在 foreach 里改集合，会抛异常 */ }

// 正确写法 1：倒序遍历
for (int i = nums.Count - 1; i >= 0; i--)
{
    if (nums[i] == 2) nums.RemoveAt(i);
}
// 正确写法 2：用 RemoveAll
nums.RemoveAll(n => n == 2);
```

### 2. Dictionary<TKey, TValue>（字典）

通过 key 快速查 value，查找速度接近 O(1)。

```csharp
Dictionary<string, int> dict = new Dictionary<string, int>();
dict["apple"] = 5;                  // 索引赋值（key 不存在会新增）
dict.Add("banana", 3);              // Add 方法

// 访问
int a = dict["apple"];              // → 5（key 不存在会抛 KeyNotFoundException）
bool ok = dict.TryGetValue("pear", out int v);  // ok → False，v → 0

// 常用方法
bool has = dict.ContainsKey("apple");   // → True
dict.Remove("banana");                  // 删除
Console.WriteLine(dict.Count);          // → 1

// 遍历
foreach (KeyValuePair<string, int> kvp in dict)
{
    Console.WriteLine($"{kvp.Key} = {kvp.Value}");   // apple = 5
}
```

**坑：** `Add` 重复 key 会抛异常，但索引赋值 `dict["key"] = v` 不会（会覆盖）。

```csharp
dict.Add("apple", 10);    // 若 apple 已存在 → ArgumentException
dict["apple"] = 10;       // 安全，覆盖原值
```

### 3. Queue<T>（队列，先进先出 FIFO）

像排队买饭，先来的先服务。

```csharp
Queue<string> queue = new Queue<string>();
queue.Enqueue("A");       // 入队
queue.Enqueue("B");
Console.WriteLine(queue.Peek());   // 查看队首 → A（不移除）
string first = queue.Dequeue();    // 出队 → A
Console.WriteLine(queue.Count);    // → 1
```

### 4. Stack<T>（栈，后进先出 LIFO）

像叠盘子，最后放的最先拿。

```csharp
Stack<int> stack = new Stack<int>();
stack.Push(1);             // 入栈
stack.Push(2);
Console.WriteLine(stack.Peek());   // 查看栈顶 → 2
int top = stack.Pop();     // 出栈 → 2
Console.WriteLine(stack.Count);    // → 1
```

### 5. HashSet<T>（集合，无重复）

自动去重，可以做集合运算。

```csharp
HashSet<int> setA = new HashSet<int> { 1, 2, 3, 4 };
HashSet<int> setB = new HashSet<int> { 3, 4, 5, 6 };

setA.Add(1);          // 已存在，不会重复添加，Count 不变
bool has = setA.Contains(2);   // → True

// 集合运算（会修改 setA 本身）
// setA.UnionWith(setB);       // 并集：{1,2,3,4,5,6}
// setA.IntersectWith(setB);   // 交集：{3,4}
// setA.ExceptWith(setB);      // 差集：{1,2}
```

### 6. 集合对比表

| 集合 | 特点 | 主要用途 | 访问复杂度 |
|------|------|----------|------------|
| List<T> | 有序、可重复、可变长 | 通用动态数组 | 索引 O(1)，查找 O(n) |
| Dictionary<K,V> | key-value、key 唯一 | 快速查找 | O(1) |
| Queue<T> | FIFO | 任务排队、BFS | 入出 O(1) |
| Stack<T> | LIFO | 撤销、DFS、表达式求值 | 入出 O(1) |
| HashSet<T> | 无重复、无序 | 去重、集合运算 | O(1) |

---

## 二、泛型

### 1. 为什么用泛型

- **类型安全**：编译期就能发现类型错误
- **避免装箱拆箱**：值类型存到 `ArrayList` 要装箱，存到 `List<int>` 不用
- **代码复用**：一份代码适用多种类型

### 2. 泛型类

```csharp
class MyList<T>
{
    public T[] items = new T[10];
    public int Count { get; private set; }

    public void Add(T item)
    {
        items[Count++] = item;
    }

    public T Get(int index) => items[index];
}

// 使用
MyList<int> list = new MyList<int>();
list.Add(10);
Console.WriteLine(list.Get(0));   // → 10
```

### 3. 泛型方法

```csharp
T GetFirst<T>(T[] arr)
{
    if (arr == null || arr.Length == 0) return default(T);
    return arr[0];
}

int n = GetFirst<int>(new[] { 5, 6, 7 });   // → 5
string s = GetFirst(new[] { "hi", "yo" });   // 类型推断 → hi
```

### 4. 泛型约束（where）

约束告诉编译器 T 必须满足什么条件。

```csharp
// 引用类型
class A<T> where T : class { }

// 值类型
class B<T> where T : struct { }

// 有无参公共构造
class C<T> where T : new()
{
    public T Create() => new T();
}

// 实现某接口
class D<T> where T : IComparable<T>
{
    public T Max(T a, T b) => a.CompareTo(b) >= 0 ? a : b;
}

// 继承自某类
class E<T> where T : Animal { }

// 多约束（同时满足）
class F<T> where T : class, IComparable<T>, new() { }
```

### 5. 常见泛型集合

`List<T>`、`Dictionary<TKey, TValue>`、`Queue<T>`、`Stack<T>`、`HashSet<T>`（前面已介绍）。

### 6. 坑

**泛型类型参数不能直接实例化**，除非加了 `new()` 约束：

```csharp
class Wrong<T>
{
    // public T item = new T();   // 编译错误！T 不一定是可实例化的
}

class Right<T> where T : new()
{
    public T item = new T();      // 正确，因为 new() 约束保证有无参构造
}
```

---

## 三、委托和事件

### 1. 委托（delegate）

委托就是方法的"引用"，可以把方法当参数传递，类似 C 的函数指针但类型安全。

```csharp
// 声明委托类型
delegate void MyDelegate(string msg);

void SayHello(string msg) => Console.WriteLine($"Hello, {msg}");

// 使用
MyDelegate d = SayHello;
d("World");   // → Hello, World
```

**多播委托**（一个委托调多个方法）：

```csharp
void Method1(string m) => Console.WriteLine($"M1: {m}");
void Method2(string m) => Console.WriteLine($"M2: {m}");

MyDelegate d = Method1;
d += Method2;          // 添加
d("Hi");               // → M1: Hi  \n  M2: Hi
d -= Method1;          // 移除
```

**坑：** 多播委托若没赋值直接调用会 `NullReferenceException`，用 `?.Invoke()`：

```csharp
MyDelegate d = null;
// d("x");            // NullReferenceException
d?.Invoke("x");       // 安全，null 时不调用
```

### 2. 内置委托类型

C# 内置了三种常用委托，省得自己声明。

```csharp
// Action：无返回值（最多 16 个参数）
Action<string> log = msg => Console.WriteLine(msg);
log("hi");                         // → hi

// Func：有返回值（最后一个参数是返回类型）
Func<int, int> square = x => x * x;
Console.WriteLine(square(5));      // → 25

Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4));      // → 7

// Predicate：返回 bool，常用作查找条件
Predicate<int> isEven = n => n % 2 == 0;
Console.WriteLine(isEven(4));      // → True
```

### 3. Lambda 表达式

`=>` 读作"goes to"，是匿名方法的简写。

```csharp
// 表达式形式
Func<int, int> square = x => x * x;

// 语句块形式
Func<int, int> abs = x =>
{
    if (x < 0) return -x;
    return x;
};

// 在集合方法中使用
List<int> nums = new List<int> { 3, 8, 1, 6, 9 };
int firstBig = nums.Find(x => x > 5);          // → 8
List<int> bigs = nums.FindAll(x => x > 5);     // → [8, 6, 9]
```

### 4. 事件（event）

事件是特殊的委托，**限制外部只能 `+=` 和 `-=`，不能直接赋值或触发**，更安全。

```csharp
class Button
{
    // 声明事件
    public event EventHandler OnClick;

    public void Click()
    {
        Console.WriteLine("按钮被点击");
        // 触发事件（用 ?.Invoke 防止无订阅者）
        OnClick?.Invoke(this, EventArgs.Empty);
    }
}

// 使用
Button btn = new Button();
btn.OnClick += (sender, e) => Console.WriteLine("处理点击 1");
btn.OnClick += (sender, e) => Console.WriteLine("处理点击 2");
btn.Click();
// 输出：
// 按钮被点击
// 处理点击 1
// 处理点击 2

// btn.OnClick = null;     // 编译错误！事件外部不能赋值
// btn.OnClick(this, ..);  // 编译错误！事件外部不能触发
```

**vs 委托：** 事件更安全，外部不能随意触发或清空。

**坑：** 忘记取消订阅可能导致内存泄漏（订阅者被事件持有，无法被 GC 回收）：

```csharp
// 在对象销毁前应取消订阅
btn.OnClick -= handler;
```

---

## 四、LINQ（Language Integrated Query）

LINQ 让你像写 SQL 一样查询各种数据源。

### 1. 查询语法

```csharp
List<int> nums = new List<int> { 3, 8, 1, 6, 9, 2 };

var result = from x in nums
             where x > 5
             orderby x
             select x;

foreach (var n in result) Console.WriteLine(n);   // 6 8 9
```

### 2. 方法语法（更常用）

```csharp
// 过滤
var bigs = nums.Where(x => x > 5);                 // [8, 6, 9]

// 投影
var doubled = nums.Select(x => x * 2);             // [6,16,2,12,18,4]

// 排序
var asc = nums.OrderBy(x => x);                    // 1,2,3,6,8,9
var desc = nums.OrderByDescending(x => x);         // 9,8,6,3,2,1

// 取第一个
int first = nums.First(x => x > 5);                // 8（找不到抛异常）
int firstOr = nums.FirstOrDefault(x => x > 100);   // 0（找不到返回默认值）

// 判断
bool anyBig = nums.Any(x => x > 5);                // True
bool allBig = nums.All(x => x > 0);                // True
int count = nums.Count(x => x > 5);                // 3

// 聚合
int sum = nums.Sum();                              // 29
int max = nums.Max();                              // 9
int min = nums.Min();                              // 1
double avg = nums.Average();                       // 4.833...

// 去重 / 截取 / 分组
var distinct = nums.Distinct();
var top3 = nums.OrderByDescending(x => x).Take(3);  // 取前 3 大
var skip2 = nums.Skip(2);                            // 跳过前 2 个
var grouped = nums.GroupBy(x => x % 2);              // 按奇偶分组
```

### 3. 综合示例

```csharp
class Student
{
    public string Name { get; set; }
    public int Score { get; set; }
}

List<Student> students = new List<Student>
{
    new Student { Name = "小明", Score = 85 },
    new Student { Name = "小红", Score = 92 },
    new Student { Name = "小刚", Score = 78 },
    new Student { Name = "小华", Score = 92 },
};

// 查分数大于 80 的学生，按分数降序，取名字
var topNames = students
    .Where(s => s.Score > 80)
    .OrderByDescending(s => s.Score)
    .ThenBy(s => s.Name)
    .Select(s => s.Name)
    .ToList();

// topNames → ["小红", "小华", "小明"]
```

### 4. 坑

**LINQ 是延迟执行的**，只有在 `ToList()`、`ToArray()`、`foreach` 等真正取值时才执行：

```csharp
var query = nums.Where(x => { Console.WriteLine($"检查 {x}"); return x > 5; });
// 此时还没输出，因为 query 还没被消费
var list = query.ToList();   // 此时才输出"检查 ..."
```

---

## 五、多线程

### 1. Thread（传统线程）

```csharp
Thread t = new Thread(() =>
{
    for (int i = 0; i < 3; i++) Console.WriteLine($"工作线程: {i}");
});
t.Start();
t.Join();   // 等待线程结束
Console.WriteLine("主线程结束");
```

**坑：** 线程不能直接访问 UI 控件（WinForm/WPF 中要用 `Control.Invoke` 或 `Dispatcher.Invoke`）。

### 2. Task（推荐）

基于线程池，比 Thread 更轻量、更好管理。

```csharp
// 启动任务
Task task = Task.Run(() =>
{
    Console.WriteLine("Task 执行中");
});

// 带返回值
Task<int> task2 = Task.Run(() => 42);
int result = task2.Result;   // 阻塞等结果 → 42

// 延续任务
Task.Run(() => 1)
    .ContinueWith(t => Console.WriteLine(t.Result + 1));   // → 2
```

### 3. async/await（推荐）

异步编程的现代写法，代码看起来像同步，但不会阻塞线程。

```csharp
async Task<string> DownloadAsync()
{
    await Task.Delay(1000);          // 模拟耗时操作
    return "下载完成";
}

async Task DoWork()
{
    Console.WriteLine("开始");
    string data = await DownloadAsync();
    Console.WriteLine(data);          // → 下载完成
    Console.WriteLine("结束");
}
```

**规则：**
- `async` 方法必须至少有一个 `await`（否则编译警告）
- 返回类型一般是 `Task` 或 `Task<T>`，事件处理器可用 `async void`

**坑 1：** `async void` 只能用于事件处理器，其他场景用 `async Task`（异常无法捕获、无法等待）。

**坑 2：** `await` 只能在 `async` 方法中使用：

```csharp
// void Wrong() { await Task.Delay(100); }   // 编译错误
async Task Right() { await Task.Delay(100); }
```

**坑 3：** 不要用 `.Result` 或 `.Wait()` 等 `async` 方法，会死锁，要用 `await`。

### 4. 并发问题

多线程访问共享数据要加锁。

```csharp
int counter = 0;
object locker = new object();

// 不加锁：结果可能不是 2000000
for (int i = 0; i < 2; i++)
{
    Task.Run(() =>
    {
        for (int j = 0; j < 1000000; j++)
        {
            lock (locker)   // 临界区，同一时刻只能一个线程进入
            {
                counter++;
            }
        }
    });
}
```

### 5. Parallel 和 PLINQ（简述）

```csharp
// 并行执行循环
Parallel.For(0, 10, i =>
{
    Console.WriteLine($"处理 {i}");
});

// PLINQ：让 LINQ 并行执行
var result = Enumerable.Range(0, 1000)
    .AsParallel()
    .Where(x => x % 2 == 0)
    .ToList();
```

---

## 六、反射和特性

### 1. 反射

反射让你在**运行时**获取类型信息、创建对象、调用方法——非常强大但性能较低。

```csharp
class Person
{
    public string Name;
    public int Age;
    public void SayHi() => Console.WriteLine($"Hi, I'm {Name}");
}

// 获取 Type
Type t = typeof(Person);
// 或：Type t = new Person().GetType();

// 获取字段
foreach (var f in t.GetFields())
    Console.WriteLine($"字段: {f.Name}");   // 字段: Name  字段: Age

// 获取方法
foreach (var m in t.GetMethods())
    Console.WriteLine($"方法: {m.Name}");

// 创建对象
object obj = Activator.CreateInstance(t);
// 给字段赋值
t.GetField("Name").SetValue(obj, "小明");
// 调用方法
t.GetMethod("SayHi").Invoke(obj, null);   // → Hi, I'm 小明
```

### 2. 特性（Attribute）

特性是给代码元素加的"标签"，运行时通过反射读取。

**内置特性：**

```csharp
[Obsolete("请使用 NewMethod()", error: false)]
void OldMethod() { }
// 调用 OldMethod 会编译警告
```

**自定义特性：**

```csharp
// 1. 定义特性
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
class MyAttribute : Attribute
{
    public string Description { get; set; }
    public MyAttribute(string desc) { Description = desc; }
}

// 2. 使用特性
[My("这是一个测试类")]
class TestClass
{
    [My("测试方法")]
    public void DoWork() { }
}

// 3. 通过反射读取特性
Type type = typeof(TestClass);
object[] attrs = type.GetCustomAttributes(typeof(MyAttribute), false);
foreach (MyAttribute attr in attrs)
{
    Console.WriteLine(attr.Description);   // → 这是一个测试类
}
```

---

## 七、文件操作（IO）

### 1. File 类

```csharp
// 写
File.WriteAllText("test.txt", "你好\n世界");
File.WriteAllLines("lines.txt", new[] { "第一行", "第二行" });

// 读
string content = File.ReadAllText("test.txt");   // → "你好\n世界"
string[] lines = File.ReadAllLines("lines.txt"); // → ["第一行", "第二行"]

// 判断
bool exists = File.Exists("test.txt");   // → True
File.Delete("test.txt");
```

### 2. Directory 类

```csharp
Directory.CreateDirectory("MyDir");
bool exists = Directory.Exists("MyDir");
string[] files = Directory.GetFiles("MyDir", "*.txt");   // 获取目录下文件
Directory.Delete("MyDir", recursive: true);              // 递归删除
```

### 3. Path 类

```csharp
string full = Path.Combine("folder", "sub", "file.txt");  // → folder\sub\file.txt
string name = Path.GetFileName(@"C:\a\b.txt");            // → b.txt
string ext = Path.GetExtension(@"C:\a\b.txt");            // → .txt
string dir = Path.GetDirectoryName(@"C:\a\b.txt");        // → C:\a
```

### 4. StreamReader / StreamWriter

流式读写，适合大文件。

```csharp
// 写
using (StreamWriter sw = new StreamWriter("log.txt"))
{
    sw.WriteLine("第一行");
    sw.WriteLine("第二行");
}

// 读
using (StreamReader sr = new StreamReader("log.txt"))
{
    string line;
    while ((line = sr.ReadLine()) != null)
        Console.WriteLine(line);
}
```

**坑：** 流用完必须释放，用 `using` 自动调用 `Dispose`，否则文件被占用。

---

## 八、高级排序

### 1. 插入排序

**思想：** 把每个元素插入到前面已排好的部分中的正确位置（像整理扑克牌）。

时间复杂度 O(n²)，但对接近有序的数据效率很高（接近 O(n)）。

```csharp
void InsertionSort(int[] arr)
{
    for (int i = 1; i < arr.Length; i++)
    {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key)
        {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

int[] a = { 5, 2, 8, 1, 9 };
InsertionSort(a);
Console.WriteLine(string.Join(", ", a));   // → 1, 2, 5, 8, 9
```

### 2. 希尔排序

**思想：** 分组插入排序，逐步缩小间隔 gap，是插入排序的改进版。

时间复杂度约 O(n^1.3)，比直接插入快很多。

```csharp
void ShellSort(int[] arr)
{
    int n = arr.Length;
    for (int gap = n / 2; gap > 0; gap /= 2)
    {
        for (int i = gap; i < n; i++)
        {
            int key = arr[i];
            int j = i;
            while (j >= gap && arr[j - gap] > key)
            {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = key;
        }
    }
}

int[] a = { 8, 1, 4, 9, 6, 3, 5, 2, 7, 0 };
ShellSort(a);
Console.WriteLine(string.Join(", ", a));   // → 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
```

### 3. 快速排序

**思想：** 选一个基准值 pivot，比它小的放左边，大的放右边，递归处理两边。

平均 O(n log n)，最坏 O(n²)（已排序数据 + 选首元素为基准时）。

```csharp
void QuickSort(int[] arr, int left, int right)
{
    if (left >= right) return;

    int pivot = arr[left];
    int i = left, j = right;
    while (i < j)
    {
        while (i < j && arr[j] >= pivot) j--;
        arr[i] = arr[j];
        while (i < j && arr[i] <= pivot) i++;
        arr[j] = arr[i];
    }
    arr[i] = pivot;

    QuickSort(arr, left, i - 1);
    QuickSort(arr, i + 1, right);
}

int[] a = { 6, 2, 8, 1, 9, 3 };
QuickSort(a, 0, a.Length - 1);
Console.WriteLine(string.Join(", ", a));   // → 1, 2, 3, 6, 8, 9
```

### 4. 归并排序

**思想：** 分治法，分成两半分别排序，再把两个有序子数组合并。

稳定排序，时间复杂度始终 O(n log n)，需要额外 O(n) 空间。

```csharp
void MergeSort(int[] arr, int left, int right)
{
    if (left >= right) return;
    int mid = (left + right) / 2;
    MergeSort(arr, left, mid);
    MergeSort(arr, mid + 1, right);
    Merge(arr, left, mid, right);
}

void Merge(int[] arr, int left, int mid, int right)
{
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right)
    {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    for (int t = 0; t < temp.Length; t++)
        arr[left + t] = temp[t];
}

int[] a = { 7, 4, 2, 9, 1, 5 };
MergeSort(a, 0, a.Length - 1);
Console.WriteLine(string.Join(", ", a));   // → 1, 2, 4, 5, 7, 9
```

### 5. 排序算法对比表

| 算法 | 平均时间 | 最坏时间 | 空间复杂度 | 稳定性 | 特点 |
|------|----------|----------|------------|--------|------|
| 冒泡 | O(n²) | O(n²) | O(1) | 稳定 | 最简单，效率最低 |
| 选择 | O(n²) | O(n²) | O(1) | 不稳定 | 交换次数少 |
| 插入 | O(n²) | O(n²) | O(1) | 稳定 | 对接近有序数据高效 |
| 希尔 | O(n^1.3) | O(n²) | O(1) | 不稳定 | 插入排序改进 |
| 快速 | O(n log n) | O(n²) | O(log n) | 不稳定 | 实际最快，常用 |
| 归并 | O(n log n) | O(n log n) | O(n) | 稳定 | 稳定且最坏也是 O(n log n) |

> **稳定**指相等元素排序后相对顺序不变。

---

## 九、其他重要概念

### 1. 装箱和拆箱

```csharp
// 装箱：值类型 → object（在堆上创建副本）
int x = 5;
object o = x;       // 装箱

// 拆箱：object → 值类型
int n = (int)o;     // 拆箱 → 5

// 装拆箱的性能陷阱
ArrayList list = new ArrayList();
for (int i = 0; i < 1000; i++)
    list.Add(i);    // 每次都装箱！
// 用 List<int> 替代，避免装箱
```

### 2. 可空类型（Nullable）

让值类型也能为 null，常用于数据库字段、未初始化状态。

```csharp
int? x = null;              // int? 等价于 Nullable<int>
Console.WriteLine(x.HasValue);   // → False
// Console.WriteLine(x.Value);   // 抛 InvalidOperationException

int? y = 10;
Console.WriteLine(y.Value);      // → 10

// 空合并运算符 ??：null 时用默认值
int z = x ?? 0;                  // → 0
int w = y ?? 0;                  // → 10
```

### 3. 元组

轻量级数据组合，不用专门定义类。

```csharp
// 匿名元组
(int, string) pair = (1, "hello");
Console.WriteLine(pair.Item1);   // → 1
Console.WriteLine(pair.Item2);   // → hello

// 命名元组（推荐）
(int id, string name) p = (1, "小明");
Console.WriteLine(p.id);         // → 1
Console.WriteLine(p.name);       // → 小明

// 方法返回多个值
(string, int) GetInfo() => ("小明", 18);
var info = GetInfo();
Console.WriteLine($"{info.Item1} {info.Item2}岁");   // → 小明 18岁
```

### 4. 模式匹配（C# 7.0+）

```csharp
// is 类型匹配
object obj = 42;
if (obj is int n && n > 0)
{
    Console.WriteLine($"正整数: {n}");   // → 正整数: 42
}

// switch 表达式（C# 8.0+）
string GetLevel(int score) => score switch
{
    >= 90 => "优秀",
    >= 80 => "良好",
    >= 60 => "及格",
    _     => "不及格"
};
Console.WriteLine(GetLevel(85));   // → 良好
```

### 5. record 类型（C# 9.0+）

不可变数据类型，自动生成 `Equals`、`GetHashCode`、`ToString`，特别适合做数据传输对象 (DTO)。

```csharp
record Person(string Name, int Age);

var p1 = new Person("小明", 20);
var p2 = new Person("小明", 20);
Console.WriteLine(p1 == p2);          // → True（基于值比较，普通类会是 False）
Console.WriteLine(p1);                // → Person { Name = 小明, Age = 20 }

// with 表达式：基于原对象创建副本，修改部分字段
var p3 = p1 with { Age = 21 };
Console.WriteLine(p3.Age);            // → 21
Console.WriteLine(p1.Age);            // → 20（原对象不变）
```

---

## 顶级篇总结

### 每个知识点一句话总结

- **集合**：List 动态数组、Dictionary 键值查找、Queue/Stack 各管 FIFO/LIFO、HashSet 去重
- **泛型**：类型安全 + 避免装箱 + 代码复用，约束限定 T 的能力
- **委托**：方法的引用，多播委托可一调多，用 `?.Invoke()` 防 null
- **事件**：受限的委托，外部只能订阅不能触发，记得取消订阅防泄漏
- **LINQ**：查询语法 + 方法语法，延迟执行，`ToList()` 才真正执行
- **多线程**：Task + async/await 是现代首选，共享数据要加 lock
- **反射**：运行时探查类型，特性是可被反射读取的标签
- **文件 IO**：File/Directory/Path 够用，大文件用流，记得 using
- **排序**：插入适合近似有序，希尔分组，快排最快，归并稳定
- **其他**：装箱拆箱有开销用泛型避免，可空类型处理空值，元组返回多值，record 做不可变数据

### 学习建议

1. **多写代码**：每个 API 都亲手跑一遍，看输出和文档对得上
2. **理解原理**：委托不只是语法糖，理解它和事件的发布订阅模式
3. **小项目练手**：写一个任务管理器、文件搜索工具、爬虫，把集合、LINQ、多线程、IO 全用上
4. **看官方文档**：[Microsoft Learn](https://learn.microsoft.com/dotnet/csharp/) 是最权威的

### 推荐进阶方向

- **ASP.NET Core**：Web 后端开发，REST API、MVC、EF Core、依赖注入
- **Unity**：C# 游戏开发，2D/3D、物理引擎、Shader
- **WPF / WinForms / MAUI**：桌面 / 跨平台客户端开发
- **Xamarin / .NET MAUI**：移动端跨平台
- **微服务**：Docker、gRPC、消息队列、可观测性

> C# 是一门优雅强大的语言，生态完善。掌握顶级篇后，你已经能写出专业级程序——选一个方向深入，做出真正能用的东西，才是成长的关键。加油！🚀

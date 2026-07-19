---
title: C#学习【核心篇】
published: 2026-07-17
pinned: false
description: "此文档记录了C#入门的核心内容"
image: "./Csharp_Image/Intermediate.avif"
tags: ["编程","C#"]
category: 学习记录
slug: Csharp_P3
---


# C# 核心篇

> 本篇覆盖面向对象编程（OOP），学完能写出结构化的面向对象程序。

---

## 一、面向对象基础

### 1. 什么是面向对象

把程序里的东西想象成"对象"，每个对象有**属性**（数据）和**行为**（方法）。

**vs 面向过程**：

| 思想 | 关注点 | 例子 |
|------|--------|------|
| 面向过程 | "怎么做"（步骤） | 把大象装冰箱：开门→装进去→关门 |
| 面向对象 | "谁来做"（对象） | 大象自己走、冰箱自己开关门 |

### 2. 三大特性

- **封装**：隐藏内部细节，只暴露必要接口
- **继承**：子类复用父类的代码
- **多态**：同一个方法，不同对象有不同表现

### 3. 类和对象

- **类** = 模板/图纸；**对象** = 实例/产品
- 类是**引用类型**！赋值是"共享"不是"复制"

```csharp
class Dog
{
    public string Name;
    public void Bark()
    {
        Console.WriteLine($"{Name}: 汪汪！");
    }
}

// 基本创建
Dog d = new Dog();
d.Name = "旺财";
d.Bark();                       // 输出: 旺财: 汪汪！

// 对象初始化器（不用写一堆构造）
Dog d2 = new Dog { Name = "小黑" };
d2.Bark();                       // 输出: 小黑: 汪汪！

// 引用类型：赋值是共享同一对象
Dog d3 = d;
d3.Name = "大黄";
Console.WriteLine(d.Name);       // 输出: 大黄（d 和 d3 是同一个对象）
```

**this 关键字**：指当前对象，常用来区分字段和参数同名。

```csharp
class Person
{
    private string name;
    public Person(string name)
    {
        this.name = name;        // this.name 是字段，name 是参数
    }
}
```

---

## 二、类的成员

### 1. 字段和访问修饰符

- **字段**：类里定义的变量，有默认值（int=0、bool=false、引用=null）
- **原则**：字段设 private，通过 public 属性/方法控制

**访问修饰符表**：

| 修饰符 | 访问范围 |
|--------|----------|
| `public` | 任何地方都能访问 |
| `private` | 只有类自己内部 |
| `protected` | 自己 + 子类 |
| `internal` | 同一程序集（项目）内 |
| `protected internal` | 同程序集 或 子类 |
| `private protected` | 同程序集内的子类 |

> 默认：类是 `internal`，类的成员是 `private`。

```csharp
class Student
{
    private int age;              // 私有字段，默认值 0
    public string Name = "未命名"; // 公有字段
}

Student s = new Student();
// s.age = 10;                    // 错误！private 外部不能访问
Console.WriteLine(s.Name);        // 输出: 未命名
```

### 2. 成员方法

- **实例方法**：需要 new 对象，`对象.方法()`
- **静态方法**（static）：不用 new，`类名.方法()`
- **静态字段**：所有对象共享一份

```csharp
class Counter
{
    private int count = 0;        // 实例字段
    public static int TotalCount = 0; // 静态字段，所有对象共享

    public void Increment()       // 实例方法
    {
        count++;
        TotalCount++;
    }

    public static int GetTotal()  // 静态方法
    {
        return TotalCount;
        // return count;          // 错误！静态方法不能访问实例字段
    }
}

Counter c1 = new Counter();
Counter c2 = new Counter();
c1.Increment();
c2.Increment();
c2.Increment();
Console.WriteLine(Counter.TotalCount); // 输出: 3（共享）
Console.WriteLine(Counter.GetTotal()); // 输出: 3
```

> ⚠️ **坑**：静态方法不能访问实例成员（因为没有 this，没有对象上下文）。

### 3. 属性

属性 = 字段的"安全包装"。`get` 读、`set` 写，`value` 是 set 的隐式参数。

```csharp
class Person
{
    private int age;              // 私有字段
    public int Age                // 带验证的属性
    {
        get { return age; }
        set
        {
            if (value < 0 || value > 150)
                age = 0;          // value 是隐式参数
            else
                age = value;
        }
    }

    public string Name { get; set; }      // 自动属性（最常用）
    public string Id { get; } = "0001";   // 只读属性（可在构造里写）
    public string Info                     // 计算属性
    {
        get { return $"{Name}, {Age}岁"; }
    }
}

Person p = new Person();
p.Name = "张三";
p.Age = 25;
Console.WriteLine(p.Info);        // 输出: 张三, 25岁
p.Age = 200;                      // 验证失败
Console.WriteLine(p.Age);         // 输出: 0
```

`{ get; private set; }`：外部只读、内部可写。

```csharp
class Account
{
    public int Money { get; private set; }
    public void Earn(int x) { Money += x; }
}

Account a = new Account();
a.Earn(100);
Console.WriteLine(a.Money);       // 输出: 100
// a.Money = 999;                 // 错误！外部不能 set
```

> ⚠️ **坑**：属性里直接调用自己 → 死循环！必须借助私有字段。
>
> ```csharp
> // ❌ 错误示范
> class Bad
> {
>     public int Age
>     {
>         get { return Age; }      // 无限递归 → 栈溢出
>         set { Age = value; }
>     }
> }
>
> // ✅ 正确做法
> class Good
> {
>     private int age;
>     public int Age
>     {
>         get { return age; }
>         set { age = value; }
>     }
> }
> ```
>
> 自动属性 `{ get; set; }` 不存在这个问题，编译器会自动生成后台字段。

### 4. 索引器

让对象像数组一样用 `[]` 访问。

```csharp
class StringCollection
{
    private string[] items = new string[10];

    public string this[int index]
    {
        get { return items[index]; }
        set { items[index] = value; }
    }

    // 可以重载（不同参数类型）
    public string this[string key]
    {
        get
        {
            foreach (var s in items)
                if (s == key) return s;
            return null;
        }
    }
}

StringCollection sc = new StringCollection();
sc[0] = "hello";
sc[1] = "world";
Console.WriteLine(sc[0]);         // 输出: hello
Console.WriteLine(sc["world"]);   // 输出: world
```

> ⚠️ **坑**：索引器**不能是 static**。适用场景：自定义集合、键值查找。

---

## 三、构造函数和析构

### 1. 构造函数

- 名字和类名一样，无返回类型，new 时自动调用
- 可以重载
- `: this(...)` 链式调用本类其他构造函数

```csharp
class Point
{
    public int X, Y;

    public Point()                // 无参构造
    {
        X = 0; Y = 0;
    }

    public Point(int x, int y)    // 带参构造
    {
        X = x; Y = y;
    }

    public Point(int x) : this(x, 0) { } // 链式调用上面的构造
}

Point p1 = new Point();
Point p2 = new Point(3, 4);
Point p3 = new Point(5);
Console.WriteLine($"{p1.X},{p1.Y}");   // 输出: 0,0
Console.WriteLine($"{p2.X},{p2.Y}");   // 输出: 3,4
Console.WriteLine($"{p3.X},{p3.Y}");   // 输出: 5,0
```

> ⚠️ **坑**：一旦你写了**带参构造**，编译器就不再自动生成无参构造了，需要时得自己补。
>
> ```csharp
> class Animal
> {
>     public string Name;
>     public Animal(string name) { Name = name; }
> }
> // Animal a = new Animal();   // 错误！没有无参构造
> ```

### 2. 析构函数

- `~类名()`，对象被 GC 回收时调用
- **不能**有参数、不能有修饰符、不能手动调用

```csharp
class Temp
{
    ~Temp()                       // 析构函数
    {
        Console.WriteLine("Temp 被回收了");
    }
}
```

> 实际开发很少用析构函数，资源释放推荐用 `IDisposable`。

### 3. 垃圾回收（GC）

- 自动管理内存，不用手动释放
- 分 3 代回收，**第 0 代最频繁**（新对象）
- `IDisposable` + `using`：需要立刻释放资源时用

```csharp
class FileReader : IDisposable
{
    public void Read()
    {
        Console.WriteLine("读取中...");
    }

    public void Dispose()
    {
        Console.WriteLine("释放资源");
    }
}

using (FileReader fr = new FileReader())
{
    fr.Read();                    // 输出: 读取中...
}                                // 离开 } 自动调用 Dispose()，输出: 释放资源
```

> ⚠️ **坑**：析构函数的执行时机**不确定**，由 GC 决定。要确定释放就用 `using`。

---

## 四、封装

封装的核心三步：

1. 字段设 `private`
2. 通过属性或方法控制读写
3. `set` 里加数据验证

**好处**：保护数据、隐藏实现、便于修改。

```csharp
class BankAccount
{
    private decimal balance;      // 私有字段，外部碰不到

    public decimal Balance        // 只读属性
    {
        get { return balance; }
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            Console.WriteLine("存款必须大于0");
        else
            balance += amount;
    }

    public bool Withdraw(decimal amount)
    {
        if (amount > balance)
        {
            Console.WriteLine("余额不足");
            return false;
        }
        balance -= amount;
        return true;
    }
}

BankAccount acc = new BankAccount();
acc.Deposit(100);
acc.Deposit(-50);                 // 输出: 存款必须大于0
acc.Withdraw(30);
Console.WriteLine(acc.Balance);   // 输出: 70
// acc.balance = 1000000;         // 错误！不能直接改
```

---

## 五、继承

### 1. 基本语法

`class 子类 : 父类 { }`，子类获得父类所有**非 private** 成员。C# **只能单继承**（一个父类）。

```csharp
class Animal
{
    public string Name;
    protected int Age;            // 子类可访问

    public Animal(string name) { Name = name; }

    public void Eat()
    {
        Console.WriteLine($"{Name}在吃东西");
    }

    public virtual void Speak()   // 虚方法，可被重写
    {
        Console.WriteLine("...");
    }
}

class Dog : Animal
{
    public Dog(string name) : base(name)  // 调用父类构造
    {
        Age = 1;                  // protected 成员子类可访问
    }

    public override void Speak()  // 重写父类方法
    {
        Console.WriteLine($"{Name}: 汪汪！");
    }
}

Dog d = new Dog("旺财");
d.Eat();                          // 输出: 旺财在吃东西
d.Speak();                        // 输出: 旺财: 汪汪！
```

### 2. base 关键字

- `: base(...)`：调用父类**构造函数**
- `base.Method()`：调用父类**方法**

```csharp
class Puppy : Dog
{
    public Puppy(string name) : base(name) { }

    public override void Speak()
    {
        base.Speak();             // 先调用父类版本：旺财: 汪汪！
        Console.WriteLine("（小奶狗声音）");
    }
}
```

### 3. protected

父类的 `protected` 成员：子类能访问，**外部不能**。

```csharp
Dog d = new Dog("旺财");
// d.Age = 2;                     // 错误！外部访问不了 protected
```

### 4. 子类构造函数

子类构造**必须**调用父类构造（默认调无参的）。父类若没有无参构造，子类**必须**用 `base` 指定。

```csharp
class Animal
{
    public string Name;
    public Animal(string name) { Name = name; }
    // 没有无参构造
}

class Cat : Animal
{
    // 不写 base 会报错！因为找不到无参父类构造
    public Cat(string name) : base(name) { }
}

Cat c = new Cat("咪咪");
Console.WriteLine(c.Name);        // 输出: 咪咪
```

> ⚠️ **坑**：子类构造函数忘了调 `base`，且父类没有无参构造 → 编译报错。

### 5. Object 祖先类

所有类都继承自 `System.Object`。常用方法：`ToString`、`Equals`、`GetType`。

```csharp
class Person
{
    public string Name;
    public Person(string name) { Name = name; }

    public override string ToString()  // 重写 Object 的方法
    {
        return $"Person: {Name}";
    }
}

Person p = new Person("张三");
Console.WriteLine(p.ToString());  // 输出: Person: 张三
Console.WriteLine(p.GetType());   // 输出: Person
Console.WriteLine(p.Equals(p));   // 输出: True
```

### 6. 密封类

`sealed class`：不能被继承。

```csharp
sealed class FinalClass { }
// class Sub : FinalClass { }     // 错误！密封类不能被继承
```

---

## 六、多态

### 1. 虚方法和重写

- 父类用 `virtual` 标记可重写的方法
- 子类用 `override` 重写
- 运行时根据**实际类型**决定调用哪个版本（这就是多态）

```csharp
class Shape
{
    public virtual double Area() { return 0; }
}

class Circle : Shape
{
    public double Radius;
    public Circle(double r) { Radius = r; }

    public override double Area()
    {
        return Math.PI * Radius * Radius;
    }
}

class Rectangle : Shape
{
    public double Width, Height;
    public Rectangle(double w, double h) { Width = w; Height = h; }

    public override double Area()
    {
        return Width * Height;
    }
}

Shape s1 = new Circle(2);
Shape s2 = new Rectangle(3, 4);
Console.WriteLine(s1.Area());     // 输出: 12.5663706143592
Console.WriteLine(s2.Area());      // 输出: 12
```

### 2. 隐藏方法（new）

子类用 `new` 隐藏父类方法（**不推荐**）。

**override vs new 对比**：

| 对比项 | `override` | `new` |
|--------|-----------|-------|
| 是否多态 | ✅ 是 | ❌ 否 |
| 父类要求 | 必须 `virtual`/`abstract` | 无要求 |
| 调用规则 | 看实际类型 | 看声明类型 |
| 推荐 | ✅ | ❌（容易混乱） |

```csharp
class Base
{
    public virtual void Show() { Console.WriteLine("Base.Show"); }
}

class DerivedA : Base
{
    public override void Show() { Console.WriteLine("DerivedA.Show"); }
}

class DerivedB : Base
{
    public new void Show() { Console.WriteLine("DerivedB.Show"); }
}

Base a = new DerivedA();
Base b = new DerivedB();
a.Show();                        // 输出: DerivedA.Show（多态）
b.Show();                        // 输出: Base.Show（只是隐藏，不触发多态）

DerivedB db = new DerivedB();
db.Show();                       // 输出: DerivedB.Show（按声明类型来）
```

### 3. 抽象类和抽象方法

- `abstract class`：**不能实例化**，只能被继承
- `abstract` 方法：只有声明没有实现，**子类必须 override**
- 抽象类可以同时有普通成员和抽象成员

```csharp
abstract class Animal
{
    public string Name;
    public Animal(string name) { Name = name; }

    public void Breathe()         // 普通方法，子类直接用
    {
        Console.WriteLine($"{Name}在呼吸");
    }

    public abstract void Speak(); // 抽象方法，无实现，子类必须 override
}

class Dog : Animal
{
    public Dog(string name) : base(name) { }

    public override void Speak()  // 必须实现
    {
        Console.WriteLine($"{Name}: 汪汪！");
    }
}

// Animal a = new Animal("x");    // 错误！抽象类不能实例化
Animal d = new Dog("旺财");
d.Breathe();                      // 输出: 旺财在呼吸
d.Speak();                        // 输出: 旺财: 汪汪！
```

> **vs 普通类 + virtual**：抽象方法**强制**子类实现；virtual 是"可选"重写。

### 4. 接口（interface）

定义行为的"约定"，本身没有实现。

```csharp
interface IFlyable
{
    void Fly();                   // 默认 public，不能加修饰符
    // 接口里不写实现
}

interface ISwimmable
{
    void Swim();
}

class Duck : IFlyable, ISwimmable // 实现多个接口（弥补单继承）
{
    public void Fly()             // ⚠️ 实现时必须加 public
    {
        Console.WriteLine("鸭子飞起来了");
    }

    public void Swim()
    {
        Console.WriteLine("鸭子游泳");
    }
}

Duck d = new Duck();
d.Fly();                          // 输出: 鸭子飞起来了
d.Swim();                         // 输出: 鸭子游泳

IFlyable f = new Duck();          // 接口引用指向实现类
f.Fly();                          // 输出: 鸭子飞起来了
```

**接口可以包含属性**：

```csharp
interface IHasName
{
    string Name { get; set; }
}

class Person : IHasName
{
    public string Name { get; set; }
}
```

> C# 8.0+ 接口可以提供**默认实现**（用 `virtual` 关键字写方法体），但初学阶段了解即可。

**接口 vs 抽象类对比表**：

| 对比项 | 接口 `interface` | 抽象类 `abstract class` |
|--------|------------------|--------------------------|
| 能否实例化 | ❌ 不能 | ❌ 不能 |
| 多继承 | ✅ 一个类可实现多个 | ❌ 只能继承一个 |
| 字段 | ❌ 不能有 | ✅ 可以有 |
| 构造函数 | ❌ 没有 | ✅ 有 |
| 方法实现 | 不能（C#8.0+ 可默认） | 可以有抽象 + 非抽象 |
| 访问修饰符 | 默认 public（不能加） | 各种都行 |
| 使用场景 | 定义"能做什么"（行为约定） | 共享代码 + 强制实现 |

### 5. 里氏替换原则

父类引用可以指向子类对象：`Animal a = new Dog();`。配合 `is` 和 `as` 做类型转换。

```csharp
class Animal { }
class Dog : Animal
{
    public void Bark() { Console.WriteLine("汪汪！"); }
}
class Cat : Animal { }

Animal a = new Dog();             // 父类引用指向子类对象

// is：判断是否是某类型
if (a is Dog)
{
    Console.WriteLine("是狗");     // 输出: 是狗
}

// as：安全转换，失败返回 null
Dog d = a as Dog;
if (d != null)
{
    d.Bark();                     // 输出: 汪汪！
}

// 模式匹配（C# 7.0+，推荐写法）
if (a is Dog dog)
{
    dog.Bark();                   // 输出: 汪汪！
}

Cat c = a as Cat;                 // 转换失败
Console.WriteLine(c == null);     // 输出: True
```

> ⚠️ **坑**：`as` 转换失败返回 null，用之前要判空；`is` 配合模式匹配最简洁。

---

## 七、静态类和静态成员

### 1. 静态类

`static class`：不能实例化，所有成员**必须**都是静态的。适合工具类。

```csharp
static class MathHelper
{
    public static double Square(double x) { return x * x; }
    public static double Cube(double x) { return x * x * x; }
}

// MathHelper m = new MathHelper(); // 错误！静态类不能实例化
Console.WriteLine(MathHelper.Square(3)); // 输出: 9
Console.WriteLine(MathHelper.Cube(2));   // 输出: 8
```

### 2. 静态成员

- 静态字段：所有对象**共享一份**
- 静态方法：通过类名调用，**不能**访问实例成员

```csharp
class User
{
    public static int UserCount = 0;   // 所有对象共享
    public string Name;

    public User(string name)
    {
        Name = name;
        UserCount++;
    }
}

new User("张三");
new User("李四");
Console.WriteLine(User.UserCount);     // 输出: 2
```

### 3. 静态构造函数

`static ClassName()`：在**第一次使用类时**自动调用，只执行一次。

- 不能有参数
- 不能有访问修饰符

```csharp
class Config
{
    public static string Version;

    static Config()               // 静态构造函数
    {
        Console.WriteLine("静态构造执行");
        Version = "1.0.0";
    }
}

Console.WriteLine("开始");
Console.WriteLine(Config.Version);
// 输出:
// 开始
// 静态构造执行（第一次访问 Config 时触发）
// 1.0.0
Console.WriteLine(Config.Version); // 输出: 1.0.0（不会再执行静态构造）
```

---

## 八、其他重要概念

### 1. 密封类和方法

- `sealed class`：不能被继承
- `sealed override`：不能被子类**再次重写**

```csharp
class Base
{
    public virtual void Show() { Console.WriteLine("Base"); }
}

class Mid : Base
{
    public sealed override void Show() { Console.WriteLine("Mid"); }
}

class Leaf : Mid
{
    // public override void Show() { }  // 错误！sealed 不能再重写
}
```

### 2. 扩展方法（简述）

在不修改原类的前提下，给现有类型"加方法"。

- 必须写在**静态类**里
- 第一个参数加 `this`

```csharp
static class StringExtensions
{
    public static void Print(this string s)   // this 表示扩展 string
    {
        Console.WriteLine(s);
    }

    public static string Repeat(this string s, int times)
    {
        string result = "";
        for (int i = 0; i < times; i++)
            result += s;
        return result;
    }
}

string msg = "Hello";
msg.Print();                     // 输出: Hello（像实例方法一样调用）
Console.WriteLine("ab".Repeat(3)); // 输出: ababab
```

### 3. partial 类（简述）

`partial class`：把一个类**拆分到多个文件**，编译时合并。常用于自动生成代码 + 手写代码分离。

```csharp
// 文件1: Person.Part1.cs
partial class Person
{
    public string Name;
    public void SayHello() { Console.WriteLine($"我是{Name}"); }
}

// 文件2: Person.Part2.cs
partial class Person
{
    public int Age;
    public void ShowAge() { Console.WriteLine($"我{Age}岁"); }
}

// 使用时是一个完整的类
Person p = new Person { Name = "张三", Age = 20 };
p.SayHello();                    // 输出: 我是张三
p.ShowAge();                     // 输出: 我20岁
```

---

## 高级篇总结

### 知识点一句话总结

| 知识点 | 一句话总结 |
|--------|------------|
| 类和对象 | 类是图纸，对象是产品；类是引用类型，赋值是共享 |
| 封装 | 字段私有，属性/方法控制读写，set 里加验证 |
| 继承 | `: 父类` 复用代码，只能单继承，子类构造必须调 base |
| 多态 | virtual + override，运行时按实际类型调方法 |
| 抽象类 | 不能实例化，abstract 方法强制子类实现 |
| 接口 | 行为约定，可多实现，实现方法必须加 public |
| 静态类 | 不能实例化，工具类专用，成员全静态 |
| 索引器 | 让对象像数组一样用 [] 访问 |
| 析构/GC | GC 自动回收，确定释放用 IDisposable + using |
| 扩展方法 | 静态类 + this 参数，给现有类型加方法 |
| partial | 一个类拆多个文件，编译时合并 |

### OOP 三大特性的关系

```
        ┌─────────────────────────────┐
        │         面向对象 OOP          │
        └─────────────────────────────┘
                     │
     ┌───────────────┼───────────────┐
     ▼               ▼               ▼
  【封装】         【继承】         【多态】
  字段私有         子类复用父类     同一方法
  属性包装         单继承机制       不同表现
     │               │               │
     │               │               │
     └─── 基础 ──────┘               │
                     │               │
                     └── 前提 ───────┘
                       继承是多态的基础
```

- **封装**是基础：保护数据，是面向对象的第一步
- **继承**是桥梁：复用代码，为多态提供前提
- **多态**是巅峰：同一接口、不同实现，是 OOP 最强大的特性

### 易错点速查

| 错误 | 说明 |
|------|------|
| 属性 get/set 调自己 | 死循环，需私有字段 |
| 写了带参构造丢了无参 | 记得手动补无参构造 |
| 静态方法访问实例字段 | 没有 this，访问不了 |
| 接口实现方法忘加 public | 默认不是 public，会报错 |
| 抽象类直接 new | 抽象类不能实例化 |
| 子类构造忘调 base | 父类没无参构造时报错 |
| new 当 override 用 | 不触发多态，按声明类型调用 |
| 用析构释放资源 | 时机不确定，改用 IDisposable |

### 学习建议

1. **先理解类和对象**：多写几个实体类（Student、Car、Book）练手感
2. **封装是基本功**：习惯把字段设 private，用属性暴露
3. **继承 + 多态一起练**：写一个 Shape 体系（Circle、Rectangle），用父类引用调 Area()
4. **接口思维**：学会用接口定义"能力"（IFlyable、ISwimmable），而不是用继承堆功能
5. **多读源码**：.NET 类库里到处是 OOP 的优秀实践（如 Stream 体系、集合体系）

> 学完本篇，你已经掌握了 C# 面向对象的核心。下一篇进入泛型、委托、LINQ 等更高级特性，加油！💪

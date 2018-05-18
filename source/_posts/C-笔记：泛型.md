---
title: C#笔记：泛型
date: 2018-05-17 21:52:46
tags: [C#,学习笔记]
categories: "C#"
copyright: true
top: 150
---
{% note primary %}
**C#学习笔记** 
**包含泛型类，泛型方法，扩展方法和泛型类**
**泛型结构，泛型委托，泛型接口**
**主要参考C#图解教程第四版**
{% endnote %}
<!-- more --> 
## 泛型类

1.类型不是对象而是对象模板，泛型类型也不是类型，而是类型的模板。
2.C#提供了五种泛型：类，结构，接口，委托和方法
3.可以从同一个泛型类型构建出很多不同的类类型，每一个都有独立的类类型，就好像它们都有独立的非泛型类型声明一样

### 泛型类举例

{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace Program
{
    class MyStack<T>
    {
        //声明泛型数组
        T[] StackArry;
        int StackPointer = 0;

        //为数组添加数据
        public void Push(T x)
        {
            //如果StackPointer小于MaxStack
            if(!IsStackFull)
            {
                StackArry[StackPointer++] = x;
            }
        }

        public T Pop()
        {
            //如果StackPointer>0,返回StackArry[--StackPointer]的值
            return (!IsStackEmpty)
                ? StackArry[--StackPointer]
                : StackArry[0];
        }

        const int MaxStack = 10;
        //如果StackPointer >= MaxStack返回真，否则返回假
        bool IsStackFull { get { return StackPointer >= MaxStack; } }
        //如果return StackPointer <= 0返回真，否则返回假
        bool IsStackEmpty { get { return StackPointer <= 0; } }

        public MyStack()
        {
            //初始化泛型数组
            StackArry = new T[MaxStack];
        }

        public void Print()
        {
            for(int i = StackPointer-1; i >= 0; i--)
            {
                Console.WriteLine("value:{0}    StackPointer={1}",StackArry[i],StackPointer);
            }
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            MyStack<int> StackInt = new MyStack<int>();
            MyStack<string> StackString = new MyStack<string>();

            StackInt.Push(3);
            StackInt.Push(5);
            StackInt.Push(7);
            StackInt.Push(9);
            //Console.WriteLine(StackInt.Pop());
            StackInt.Print();

            StackString.Push("this is fun");
            StackString.Push("hi there");
            StackString.Print();
            Console.ReadKey();
        }  
    }
}

{% endcodeblock %}

输出结果
```
value:9    StackPointer=4
value:7    StackPointer=4
value:5    StackPointer=4
value:3    StackPointer=4
value:hi there    StackPointer=2
value:this is fun    StackPointer=2

```

### 泛型与非泛型之间的区别

|  | 非泛型 | 泛型 | 
| - | :-: | -: | 
| 源代码大小 | 更大：我们需要为每一种类型编写一个新的实现| 更小：不管构造类型的数量有多少，我们只需要一个实现 | 
| 可执行大小 | 无论每一个版本中的栈是否会被使用，都会在编译的版本中出现 | 可执行文件中只会出现有构造类的类型 | 
| 写的难易度 | 易于书写，因为更加具体 | 比较难写，更抽象 |
|维护的难易度 | 更容易出问题，因为所有修改都需要应用到每一个可用类型上 | 易于维护，因为只需要修改一个地方 |


### 类型参数的约束
所有的C#对象最终都从object类继承，这些保存的项都实现了object类的成员，它包括ToString，Equals及GetType。除了这些他还不知道那些成员可用。

只要我们的代码不访问它处理的一些类型的对象（或者只要它始终是object类型的成员）， 泛型类就可以处理任何类型。符合约束的类型参数叫做未绑定的类型参数（unbounded type parameter)。然而，如果代码尝试使用其他成员，编译器会产生一个错误消息。

例如，如下代码声明了一个叫做Simple的类，它有一个叫做LessThan的方法，接受了两个泛类型的变量。LessThan尝试用小于运算符返冋结果。但是由于不是所有的类都实现了小于运算符，也就不能用任何类来代替T,所以编译器会产生一个错误消息

{% codeblock lang:C# %}
class Simple<T>
{
    static public bool LessThan(T i1,T i2)
    {
        return i1 < i2;
    }
}
{% endcodeblock %}

### 约束where

要让泛型变的更有用，需要提供额外的信息让编译器知道参数可以接受那些类型。这些额外的信息叫约束。
约束使用Where子句列出
1.每一个约束的类型参数有自己的where子句
2.如果形参有多个约束，他们在where子句中使用逗号分隔

where子句语法如下
```
where TypeParam : constraint,constraint, ……
```

例如，如下泛型类有三个类型参数，T1时未绑定的，对于T2,只有Customer类型或从Customer继承的类型才能用作类型实参，而对于T3，只有实现IComparablr的接口的类才能用于类型实参
{% codeblock lang:C# %}
    class MyClass<T1,T2,T3>
        where T2:Customer
        where T3:IComparable
    {
        ......
    }

{% endcodeblock %}

约束类型和次序
    共有五种类型的约束

| 约束类型 | 描述 | 
| - | -: | 
| 类名 | 只有这个类型的类或从他继承的类才能作类型实参 | 
| class | 任何引用类型，包括类，数组，接口，委托都可以用作类型实参 | 
| struct | 任何值类型都可以作类型实参 |
| 接口名 | 只有这个接口或实现这个接口的类型才能用作类型实参 |
| new() | 任何带有无参公共构造函数的类型都可以用作类型实参，这叫做构造函数约束 |

where子句可以以任意次序列出。然而，where子句的约束必须有特定的次序
最多只能有一个主约束，如果有则必须放在第一位。
可以有任意多的接口约束
如果存在构造函数约束，则必须放在最后。

### 附：几种常用泛型参数约束
|  | 描述 |
| - | -: | 
| T：struct | 类型参数必须是值类型。可以指定除以外的任何值 类型。 | 
| T：class | 类型参数必须是引用类型；这一点也适用于任何类、 接口、委托或数组类型 | 
| T：new() | 类型参数必须具有无参数的公共构造函数。当与其 他约束一起使用时，new() 约束必须最后指定。 |
| T：<基类名> | 类型参数必须是指定的基类或派生自指定的基类. |
| T：<接口名称> | 类型参数必须是指定的接口或实现指定的接口。可 以指定多个接口约束。约束接口也可以是泛型的。 |


## 泛型方法

泛型方法可以声明在泛型类型和非泛型类型及结构和接口中声明

{% codeblock lang:C# %}
    public void PrintData<S,T>(S p,T t)
        where S:Person
    {

    }
{% endcodeblock %}

有时编译器可以推断我们参数的类型，所以在调用时可以盛烈参数类型
如

{% codeblock lang:C# %}
int MyInt = 5;
MyMethod<int>(MyInt);
{% endcodeblock %}

改写为

{% codeblock lang:C# %}
MyMethod(MyInt);
{% endcodeblock %}

### 泛型方法举例

{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace Program
{
    class Simple
    {
        static public void ReverseAndPrint<T>(T[] arr) //泛型方法
        {
            Array.Reverse(arr);
            foreach(T item in arr)
                Console.WriteLine("{0} ,",item.ToString());
            Console.WriteLine("");
        }
    }

       
    class Program
    {
        static void Main(string[] args)
        {
            //创建不同类型
            var intArray = new int[] { 3, 5, 7, 9, 11 };
            var stringArray = new string[] { "first","second","third"};
            var doubleArray = new double[] { 3.1, 3.2, 3.3 };

            //调用方法
            Simple.ReverseAndPrint<int>(intArray);
            //推断类型并调用
            Simple.ReverseAndPrint(intArray);

            Simple.ReverseAndPrint<string>(stringArray);
            Simple.ReverseAndPrint(stringArray);

            Simple.ReverseAndPrint<double>(doubleArray);
            Simple.ReverseAndPrint(doubleArray);
            Console.ReadKey();
        }  
    }
}

{% endcodeblock %}

运行结果：
```
11 ,9 ,7 ,5 ,3 ,
3 ,5 ,7 ,9 ,11 ,

third ,second ,first ,
first ,second ,third ,

3.3 ,3.2 ,3.1 ,
3.1 ,3.2 ,3.3 ,

```

## 扩展方法和泛型类
扩展方法可以和泛型类结合使用。它允许我们将类中的静态方法关联滴哦不同的泛型类上，还允许我们像调用类构造实例的实例方法一样来调用方法。
和非泛型类一样，泛型类的扩展方法：
**必须声明为static**
**必须是静态类的成员**
**第一个参数类型中必须有关键字this，后面时扩展类型的名字**

如下代码给出了一个叫做Print的扩展方法，扩展了叫做Holder<T>的泛型类

{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;

namespace Program
{
    static class ExtendHolder
    {
        //泛型扩展方法
        public static void Print<T>(this Holder<T> h)
        {
            T[] vals = h.GetValues();
            Console.WriteLine("{0},\t{1},\t{2}",vals[0],vals[1],vals[2]);
        }
    }

    class Holder<T>
    {
        T[] Vals = new T[3];

        public Holder(T v0,T v1,T v2)
        {
            Vals[0] = v0;
            Vals[1] = v1;
            Vals[2] = v2;
        }

        public T[] GetValues()
        {
            return Vals;
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            var intHolder = new Holder<int>(3, 5, 7);
            var stringHolder = new Holder<string>("a1", "a2", "a3");
            intHolder.Print();
            stringHolder.Print();
            Console.ReadKey();
        }  
    }
}

{% endcodeblock %}

输出结果

```
3,      5,      7
a1,     a2,     a3

```

## 泛型结构
未完待续
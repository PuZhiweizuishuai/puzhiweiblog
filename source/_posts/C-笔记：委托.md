---
title: 'C#笔记：委托'
date: 2018-05-31 23:17:27
tags: [C#,学习笔记]
categories: "C#"
copyright: true
top: 150
---
{% note primary %}
**C#学习笔记** 
**委托的基本知识**
{% endnote %}
<!-- more --> 

## 委托
委托实际上是一种数据类型，我们可以使用它来定义变量。
一个委托类型的变量，可以引用任何一个满足其要求的方法（包括静态或实例方法）
委托变量有点像时一个方法的“容器”，将某一个具体的方法“装入”之后，这个委托变量就可以当成方法一样调用。

例：
{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace 委托
{
    public class MathOpt
    {
        public int Add(int x,int y)
        {
            return x + y;
        }

        public static int Max(int x,int y)
        {
            return (x > y || x == y) ? x : y;
        }
    }

    //声明一个委托方法
    public delegate int MathOptDelegate(int value1, int value2);

    class Program
    {
        static void Main(string[] args)
        {
            MathOptDelegate oppDel;
            //委托时一种用户自定义的数据类型，可以用于自定义变量
            MathOpt obj = new MathOpt();
            //oppDel = obj.Add;

            //也可以接收偶一个静态方法的引用
            oppDel = MathOpt.Max;

            //委托变量可以当成普通方法那样调用
            Console.WriteLine(oppDel(1,2));

            //可以把方法引用直接传给委托类型的参数
            Console.WriteLine(UseDelegate(obj.Add,10,20));
            Console.WriteLine(UseDelegate(MathOpt.Max,100,200));

            Console.ReadKey();
        }

        //可以定义委托类型的参数
        static int UseDelegate(MathOptDelegate option, int x, int y)
        {
            return option(x, y);
        }
    }
}
{% endcodeblock %}


## 组合委托

委托可以使用额外的运算符来组合，这个运算符会创建一个新的委托。
例如
{% codeblock lang:C# %}
MathOptDelegate oppDelA = MathOpt.Max;
MathOptDelegate oppDelB = obj.Add;
MathOptDelegate oppDelC = oppDelA + oppDelB;
{% endcodeblock %}

oppDelC成为一个具有组合调用列表的新委托。


### 为委托添加和减去方法

可以使用+=运算符为委托添加方法

例：
{% codeblock lang:C# %}
MathOptDelegate oppDelA = MathOpt.Max;
oppDelA += obj.Add;           //添加方法
{% endcodeblock %}

由于委托不可变，使用+=运算符时，实际上是创建了一个新委托


可以使用-=运算符为委托移除方法
{% codeblock lang:C# %}
oppDelA -= obj.Add;     //移除方法
{% endcodeblock %}

与增加方法一样，其实是创建了一个新委托，新委托是旧委托的副本

综合举例：

{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace MulticastDelegatelnvocationList
{
    //定义一个委托变量
    public delegate int MyDelegate(int value);
    
    public class MyClass
    {
        public int Func1(int argument)
        {
            Console.WriteLine("Func1: i={0}",argument);
            return argument;
        }

        public int Func2(int argument)
        {
            Console.WriteLine("Func2: i={0}", argument*2);
            return argument*2;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            MyClass obj = new MyClass();

            MyDelegate del1 = obj.Func1;
            del1 += obj.Func2;

            //获取方法调用列表
            Delegate[] ds = del1.GetInvocationList();  //GetInvocationList()获取方法调用列表
            Console.WriteLine("del1的委托调用列表中包含{0}个方法。",ds.GetLength(0));


            del1(5);  //先调用obj.Func1(),在调用obj.Func2()


            MyDelegate del2 = obj.Func1;
            del2 += obj.Func2;

            Console.WriteLine("del2的委托调用列表中包含{0}个方法。",del2.GetInvocationList().GetLength(0));

            del2(5);

            //组合两个委托变量
            MyDelegate mul = del1 + del2;
            ds = mul.GetInvocationList();
            Console.WriteLine("合并del1和del2后，新的委托变量mul的委托调用列表中包含{0}个方法。",ds.GetLength(0));
            int ret = mul(10);

            Console.WriteLine("合并之后，新的委托变量mul的返回值 = {0}",ret);

            //移除方法obj.Func2()
            mul -= obj.Func2;
            Console.WriteLine("移除Func2后，委托变量mul包含{0}个方法。",mul.GetInvocationList().GetLength(0));

            ret = mul(10);  //获取委托调用最后一个方法的返回值
            Console.WriteLine("移除Func2后，返回值 = {0}",ret);

            Console.ReadKey();
        }
    }
}
{% endcodeblock %}

运行结果：
```
del1的委托调用列表中包含2个方法。
Func1: i=5
Func2: i=10
del2的委托调用列表中包含2个方法。
Func1: i=5
Func2: i=10
合并del1和del2后，新的委托变量mul的委托调用列表中包含4个方法。
Func1: i=10
Func2: i=20
Func1: i=10
Func2: i=20
合并之后，新的委托变量mul的返回值 = 20
移除Func2后，委托变量mul包含3个方法。
Func1: i=10
Func2: i=20
Func1: i=10
移除Func2后，返回值 = 10

```

注：
{% note %}
如果调用列表中的方法有多个实例，-=运算符将从列表最后开始搜索，并移除第一个与方法匹配的实例。
试图删除委托中不存在的方法是没有效果的。
试图调用空委托会抛出异常，通常与null比较判断是否为空。
如果一个方法在调用列表中出现多次，当委托被调用时，每次在列表中遇到这个方法时他都会被调用一次。
如果委托有返回值并且在列表中有一个以上的方法，调用列表中最会一个方法返回的值就时委托调用的返回值，其它数值都会被忽略。
引用参数值会在调用期间发生改变。
{% endnote %}

## 另一个例子
{% codeblock lang:C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;

namespace UseTimerCallback
{
    //用于向回调函数提供参数信息
    class TaskInfo
    {
        public int count = 0;
    }


    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("敲任意键结束……");
            TaskInfo ti = new TaskInfo();
            //创建Timer对象，将一个回调函数传给它，每隔一秒调用一次
            //把ti这一个参数对象传给Timer 
            //再将ShowTime的对象引用传给它
            Timer tm = new Timer(ShowTime, ti, 0, 1000);
            Console.ReadKey();
            tm.Dispose();      //回收数据

        }
        //被回调的函数
        static void ShowTime(Object ti)
        {
            TaskInfo obj = ti as TaskInfo;  //强制类型转换
            obj.count++;
            Console.WriteLine("({0}){1}", obj.count, DateTime.Now);
        }

    }
}
{% endcodeblock %}

说明：
Timer构造函数第一个参数TimerCallback，定时引用的函数就是这么一个函数引用的
<img src="http://7xjjdc.com1.z0.glb.clouddn.com/blog/c%23/2018/5/31/timer.JPG" alt=".Net基类库Timer">
可以看出在.Net基类库中，可以明显看出TimerCallback是一个委托，返回void，接受一个object参数，同样就可以编写一个
void ShowTime(Object ti)
函数
<img src="http://7xjjdc.com1.z0.glb.clouddn.com/blog/c%23/2018/5/31/timer2.JPG" alt="TimerCallback实现">


## END
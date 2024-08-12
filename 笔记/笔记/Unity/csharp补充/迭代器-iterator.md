---
tags: live
---
关键接口：`IEnumerator, IEnumerable`
    命名空间：`using System. Collections;`
    可以通过同时继承 `IEnumerable` 和 `IEnumerator` 实现其中的方法

#### Foreach本质
1. 先获取in后面这个对象的 IEnumerator，会调用对象其中的GetEnumerator方法来获取
2. 执行得到这个IEnumerator对象中的 MoveNext方法
3. 只要MoveNext方法的返回值时true 就会去得到Current，然后复制给 item
```cs
class CustomList : IEnumerable, IEnumerator
{
    private int[] list;
    //从-1开始的光标 用于表示 数据得到了哪个位置
    private int position = -1;
    public CustomList()
    {
        list = new int[] { 1, 2, 3, 4, 5, 6, 7, 8 };
    }
    #region IEnumerable
    public IEnumerator GetEnumerator()
    {
        Reset();
        return this;
    }
    #endregion
    public object Current
    {
        get
        {
            return list[position];
        }
    }
    public bool MoveNext()
    {
        //移动光标
        ++position;
        //是否溢出 溢出就不合法
        return position < list.Length;
    }
    //reset是重置光标位置 一般写在获取 IEnumerator对象这个函数中
    //用于第一次重置光标位置
    public void Reset()
    {
        position = -1;
    }
}
class Program
    {
        static void Main(string[] args)
        {
            CustomList list = new CustomList();
        	foreach (int item in list)
            {
                Console.WriteLine(item);
            }
        }
    }
}
```
#### Yield return 语法糖
```C#
class CustomList2 : IEnumerable
{
    private int[] list;
    public CustomList2()
    {
        list = new int[] { 1, 2, 3, 4, 5, 6, 7, 8 };
    }
    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < list.Length; i++)
        {
            //yield关键字 配合迭代器使用
            //可以理解为 暂时返回 保留当前的状态 一会还会在回来
            yield return list[i];
        }
        //yield return list[0];
        //yield return list[1];
        //yield return list[2];
        //yield return list[3];
        //yield return list[4];
        //yield return list[5];
        //yield return list[6];
        //yield return list[7];
    }
}
```
#### 用 yield return 语法糖为泛型类实现迭代器
```C#
class CustomList<T> : IEnumerable
{
	private T[] array;
	public CustomList(params T[] array)
	{
		this.array = array;
	}
	public IEnumerator GetEnumerator()
	{
		for (int i = 0; i < array.Length; i++)
		{
			yield return array[i];
		}
	}
}
class Program
    {
        static void Main(string[] args)
        {
            CustomList<string> list2 = new CustomList<string>("123","321","333","555");
            foreach (string item in list2)
            {
                Console.WriteLine(item);
            }
        }
    }
}
```

#### 补充：yield return
yield return是C#中的一个关键字 ，用于简化迭代器的实现。它允许你在一个方法中逐步返回数据，而不需要显式地管理状态。使用yield return返回的数据可以用于多种场景，主要是为了简化集合的迭代和生成。
主要用途
1.	简化迭代器的实现：
•	yield return可以让你轻松地实现IEnumerable接口，而不需要显式地管理IEnumerator的状态。
2.	延迟执行：
•	yield return实现的迭代器是延迟执行的，这意味着只有在实际迭代时才会生成数据。这对于处理大数据集或需要懒加载的场景非常有用。
3.	提高代码可读性：
•	使用yield return可以使代码更加简洁和易读，减少了手动管理状态的复杂性。

以下是一些使用yield return的示例，展示了它的不同用途。
##### 示例1 ：简单的迭代器
```C#
using System;
using System.Collections;
using System.Collections.Generic;

public class SimpleCollection : IEnumerable<int>
{
    private int[] numbers = { 1, 2, 3, 4, 5 };

    public IEnumerator<int> GetEnumerator()
    {
        foreach (int number in numbers)
        {
            yield return number;
        }
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

class Program
{
    static void Main()
    {
        SimpleCollection collection = new SimpleCollection();
        foreach (int number in collection)
        {
            Console.WriteLine(number);
        }
    }
}
```
##### 示例 2：生成斐波那契数列
```C#
using System;
using System.Collections.Generic;

public class FibonacciSequence
{
    public static IEnumerable<int> GetFibonacciNumbers(int count)
    {
        int a = 0, b = 1;
        for (int i = 0; i < count; i++)
        {
            yield return a;
            int temp = a;
            a = b;
            b = temp + b;
        }
    }
}

class Program
{
    static void Main()
    {
        foreach (int number in FibonacciSequence.GetFibonacciNumbers(10))
        {
            Console.WriteLine(number);
        }
    }
}
```
##### 示例 3：延迟执行
```C#
using System;
using System.Collections.Generic;

public class DelayedExecution
{
    public static IEnumerable<int> GetNumbers()
    {
        Console.WriteLine("Start generating numbers...");
        for (int i = 1; i <= 5; i++)
        {
            yield return i;
            Console.WriteLine($"Yielded {i}");
        }
        Console.WriteLine("Finished generating numbers.");
    }
}

class Program
{
    static void Main()
    {
        IEnumerable<int> numbers = DelayedExecution.GetNumbers();
        Console.WriteLine("Before iteration");
        foreach (int number in numbers)
        {
            Console.WriteLine(number);
        }
        Console.WriteLine("After iteration");
    }
}
```
总结
•	简化迭代器实现：yield return 使得实现 IEnumerable 接口变得非常简单。
•	延迟执行：只有在实际迭代时才会生成数据，适用于处理大数据集或需要懒加载的场景。
•	提高代码可读性：减少了手动管理状态的复杂性，使代码更加简洁和易读。
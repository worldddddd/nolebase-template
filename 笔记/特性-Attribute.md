---
tags: live
---
- 特性是一种允许我们向程序的程序集添加元数据的语言结构，它是用于保存程序结构信息的某种特殊类型的类
- 特性提供功能强大的方法以将声明信息与 C# 代码（类型、方法、属性等）相关联。特性与程序实体关联后，即可在运行时使用反射查询特性信息
- 特性的目的是告诉编译器把程序结构的某组元数据嵌入程序集中，它可以放置在几乎所有的声明中（类、变量、函数等等申明）
简而言之：
	特性本质是个类
	我们可以利用特性类为元数据添加额外信息
	比如一个类、成员变量、成员方法等等为他们添加更多的额外信息
	之后可以通过反射来获取这些额外信息
### 限制自定义特性的使用范围
```cs
    //通过为特性类加特性限制其使用范围
    [AttributeUsage (AttributeTargets. Class | AttributeTargets. Struct, AllowMultiple = true, Inherited = true)]
    //参数一：AttributeTargets —— 特性能够用在哪些地方
    //参数二：AllowMultiple —— 是否允许多个特性实例用在同一个目标上
    //参数三：Inherited —— 特性是否能被派生类和重写成员继承
    Public class MyCustom2 Attribute : Attribute
    {
    }
```
### 特性的使用
基本语法：[特性名 (参数列表)]
写在类、函数、变量上一行，表示他们具有该特性信息
```cs
[MyCustom("这个是我自己写的一个用于计算的类")]
class MyClass
{
    [MyCustom("这是一个成员变量")]
    public int value;

    [MyCustom("这是一个用于计算加法的函数")]
    public void TestFun( [MyCustom("函数参数")]int a )
    {

    }
    public void TestFun(int a)
    {

    }
}
```
#### 主函数中特性的使用
```cs
class Program{
	static void Main(string[] args)
	{
		MyClass mc = new MyClass();
		Type t = mc.GetType();
		//判断是否使用了某个特性
 		//参数一：特性的类型
		//参数二：代表是否搜索继承链（属性和事件忽略此参数）
		if(t.IsDefined(typeof(MyCustomAttribute), false) )
		{
			Console.WriteLine("该类型应用了MyCustom特性");
		}
		//获取Type元数据中的所有特性
		object[] array = t.GetCustomAttributes(true);
		for(int i = 0; i < array.Length; i++)
		{
			if(array[i] is MyCustomAttribute)
			{
				Console.WriteLine((array[i] as MyCustomAttribute).info);
				//输出：这个是我自己写的一个用于计算的类
				(array[i] as MyCustomAttribute).TestFun();
			}
		}
	}
}
```

### 系统自带特性——调用者信息特性
哪个文件调用？CallerFilePath特性
哪一行调用？CallerLineNumber特性
哪个函数调用？CallerMemberName特性
需要引用命名空间 using System.Runtime.CompilerServices;
一般作为函数参数的特性
```cs
public void SpeakCaller(string str, [CallerFilePath]string fileName = "", 
    [CallerLineNumber]int line = 0, [CallerMemberName]string target = "")
{
    Console.WriteLine(str);
    Console.WriteLine(fileName);
    Console.WriteLine(line);
    Console.WriteLine(target);//一般是Main
}
```
### 系统自带特性——条件编译特性
条件编译特性：Conditional
它会和预处理指令 `#define` 配合使用
需要引用命名空间using System.Diagnostics;
主要可以用在一些调试代码上
有时想执行有时不想执行的代码
```cs
#define Fun //定义才能执行
[Conditional("Fun")]
static void Fun()
{
    Console.WriteLine("Fun执行");
}
```
### 系统自带特性——外部 Dll 包函数特性
`DllImport`：用来标记非.Net(C#)的函数，表明该函数在一个外部的DLL中定义。
一般用来调用 C或者C++的Dll包写好的方法
需要引用命名空间 `using System.Runtime.InteropServices`
```cs
[DllImport("Test.dll")]
public static extern int Add(int a, int b);
```


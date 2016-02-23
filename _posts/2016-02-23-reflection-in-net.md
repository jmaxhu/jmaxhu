---
layout: post
title: .NET反射
tags: .net, reflection
---

# .NET反射

## 简介

本文尝试以例子的方式学习 .NET 反射的各个主题. 从什么是 .NET 反射开始,介绍 [System.Reflection][reflection] 命名空间及反射中很重要的 [Type][typerel] 类. 介绍了如何得到类型信息的不同方式,还介绍了用反射的方式使用属性(propertiy)和方法(method). 最后还包括了包含动态绑定和延迟绑定等高级主题.

[reflection]: https://msdn.microsoft.com/zh-cn/library/system.reflection(v=vs.110).aspx "反射命名空间"
[typerel]: https://msdn.microsoft.com/zh-cn/library/system.type(v=vs.110).aspx "类型"
[runtime]: https://en.wikipedia.org/wiki/Run_time_(program_lifecycle_phase) "运行时"

## 什么是.NET反射?

.NET 框架的反射API允许你在[运行时][runtime] 得到类型的信息,并改变自己的结构和行为(特别是值,元数据,属性和函数). 在运行时,反射机制通过PE文件读取程序集的信息. 反射允许你使用编译时不可用的代码,比例访问类的私有变量.  它可以非常有效的得到程序集的所有类型并动态调用其中的方法. 通过这些类型,可以得到该类型包含的字段,属性,方法,事件等元素.

.NET反射API主要在 [System.Reflection][reflection] 和 [System.Type][typerel] 两个命名空间中提供.

## System.Reflection 命名空间

[System.Reflection][reflection] 命名空间包含了查看类型,方法和字段等元素类和接口.并且可以动态创建和调用它们. 下面是一些常用的类.

 类|说明
 ---|---
 Assemble|表示一个程序集，它是一个可重用、无版本冲突并且可自我描述的公共语言运行时应用程序构造块。
Module|模块
AssemblyName|完整描述程序集的唯一标识。
EventInfo|发现事件的特性并提供对事件元数据的访问权。
FieldInfo|发现字段特性并提供对字段元数据的访问权。
MemberInfo|获取有关成员特性的信息并提供对成员元数据的访问。
ParameterInfo|发现参数特性并提供对参数元数据的访问。
PropertyInfo|发现属性的特性并提供对属性元数据的访问。
MethodInfo|发现方法的特性并提供对方法元数据的访问。

在开始使用反射前, 有必要先理解 [System.Type][typerel] 类.

为了方便后续的例子演示, 我使用一个 **Car** 的类, 代码如下:

    //ICar.cs
    namespace ReflectionDemo
    {
        public interface ICar
        {
            bool IsMoving();
        }
    }

    //Car.cs
    namespace ReflectionDemo
    {
        public class Car : ICar
        {
            public Car(string color, double speed)
            {
                Color = color;
                Speed = speed;
            }

            public double Speed { get; set; }

            public string Color { get; set; }

            public void Accelerate(int accelerateBy)
            {
                Speed += accelerateBy;
            }

            public double CalculateMPG(int startMiles, int endMiles, double gallons)
            {
                return (endMiles - startMiles) / gallons;
            }

            public bool IsMoving()
            {
                return Speed > 0d;
            }
        }
    }

    //SportCar.cs
    namespace ReflectionDemo
    {
        public class SportCar : Car
        {
            public SportCar() : base("Green", 100)
            {
            }
        }
    }

## System.Type 类

[System.Type][typerel] 是实现.NET反射的主要类,是读取元数据的主要方式. System.Type是一个抽象类,表示一个 **Common Type System(CLS)** 中的一个类型.

它表示类型的定义: 类, 接口, 数组, 值, 枚举, 类型参数, 泛型等.

通过类型的成员可以得到类型的信息, 如构造器, 方法, 字段, 属性, 事件等. 以及当前类所在的模块和程序集等信息.

有3种方式得到一个类型的引用:


### 使用 System.Object.GetType()

这是一个实例方法,任何类型都继续了该方法,因为它定义在Object类中. 这种方法一般在编译时使用, 因为你必须先有一个类型的实例才能调用.例如:

    namespace ReflectionDemo
    {
        internal class Program
        {
            private static void Main(string[] args)
            {
                var car = new Car();
                var carType = car.GetType();
                Console.WriteLine(carType.FullName);

                Console.ReadLine();
            }
        }
    }

输出:

    ReflectionDemo.Car

### 使用 System.Type.GetType()

另一个更灵活的方法是使用Type的静态方法 GetType(), 调用该方法需要提供一个代码类型名称的字符串.

Type.GetType() 是一个重载方法,主要包含以下3个参数.

*  完全限定(命名空间和类型)的类型名称字符串
*  当没有找到类型时是否抛出异常
*  是否执行大小写敏感查找

示例如下:

    namespace ReflectionDemo
    {
        internal class Program
        {
            private static void Main(string[] args)
            {
                var carType = Type.GetType("ReflectionDemo.Car", true, true);
                Console.WriteLine(carType.FullName);

                Console.ReadLine();
            }
        }
    }

输出:

    ReflectionDemo.Car

### 使用 typeof () C# 操作符

最后一种得到类型信息的方法是使用 C# 的 **typeof** 操作符, 这个操作符需要传递一个类型的名称.

    namespace ReflectionDemo
    {
        internal class Program
        {
            private static void Main(string[] args)
            {
                var carType = typeof(Car);
                Console.WriteLine(carType.FullName);

                Console.ReadLine();
            }
        }
    }

输出:

    ReflectionDemo.Car

## Type类型的属性和方法

System.Type 类定义了一些属性和方法来帮助我们使用反射. 
主要属性列表:

属性|说明
---|---
Name|类型的名称
FullName|类型的包含命名空间的名称
Namespace|类型的所属命名空间
BaseType|当前类型的直接基类
IsAbstract|是否抽象类
IsArray|是否是数组
IsClass|是否是类(相对值类型)
IsEnum|是否是枚举
IsGenericTypeDefinition|是否泛型类型定义
IsGenericParameter|是否泛型参数
IsInterface|是否接口
IsPrimitive|是否原数据类型
IsPublic|是否公共的
IsSealed|是否密封的

主要方法列表:

返回类型|方法|说明
---|---|---
ConstructorInfo|GetConstructor(),GetConstructors()|取构造函数
EventInfo|GetEvent(),GetEvents()|取事件
FieldInfo|GetField(),GetFields()|取字段
InterfaceInfo|GetInterface(),GetInterfaces()|取接口
MemberInfo|GetMember(),GetMembers()|取成员
MethodInfo|GetMethod(),GetMethods()|取方法
PropertyInfo|GetProperty(),GetProperties()|取属性

使用示例:

	using System;
	using System.Text;
	using System.Reflection;

	namespace ReflectionDemo
	{
	  internal class Program
	  {
		private static void Main(string[] args)
		{
		  var carType = typeof(SportCar);
		  GetTypeProperties(carType);

		  Console.ReadLine();
		}

		public static void GetTypeProperties(Type t)
		{
		  var outputText = new StringBuilder();

		  outputText.AppendLine("Analysis of type " + t.Name);
	   	  outputText.AppendLine("Type Name: " + t.Name);
		  outputText.AppendLine("Full Name: " + t.FullName);
		  outputText.AppendLine("Namespace: " + t.Namespace);

		  Type tBase = t.BaseType;

		  if (tBase != null)
		  {
	        outputText.AppendLine("Base Type: " + tBase.Name);
		  }

		  Type underlyingSystem = t.UnderlyingSystemType;

		  if (underlyingSystem != null)
		  {
			outputText.AppendLine("UnderlyingSystem Type: " + underlyingSystem.Name);
		  }

		  outputText.AppendLine("Is Abstract Class: " + t.IsAbstract);
		  outputText.AppendLine("Is an Arry: " + t.IsArray);
		  outputText.AppendLine("Is a Class: " + t.IsClass);
		  outputText.AppendLine("Is a COM Object : " + t.IsCOMObject);

		  outputText.AppendLine("\nPUBLIC MEMBERS:");
		  var members = t.GetMembers();

		  foreach (MemberInfo nextMember in members)
		  {
			outputText.AppendLine(nextMember.DeclaringType + " " + nextMember.MemberType + "  " + nextMember.Name);
				}
		  Console.WriteLine(outputText);
		  }
		}
	}

输出:
	
    Analysis of type SportCar
    Type Name: SportCar
    Full Name: ReflectionDemo.SportCar
    Namespace: ReflectionDemo
    Base Type: Car
    UnderlyingSystem Type: SportCar
    Is Abstract Class: False
    Is an Arry: False
    Is a Class: True
    Is a COM Object : False

    PUBLIC MEMBERS:
    ReflectionDemo.Car Method  get_Speed
    ReflectionDemo.Car Method  set_Speed
    ReflectionDemo.Car Method  get_Color
    ReflectionDemo.Car Method  set_Color
    ReflectionDemo.Car Method  Accelerate
    ReflectionDemo.Car Method  CalculateMPG
    ReflectionDemo.Car Method  IsMoving
    System.Object Method  ToString
    System.Object Method  Equals
    System.Object Method  GetHashCode
    System.Object Method  GetType
    ReflectionDemo.SportCar Constructor  .ctor
    ReflectionDemo.Car Property  Speed
    ReflectionDemo.Car Property  Color

## Assembly 类
System.Reflection命名空间提供了一个**Assembly**类. 我们可以通过这个Assembly类得到程序集的信息并操作该程序集. 这个类允许我们在运行时加载模块和程序集. 这个Assembly类通过读取PE文件的得到程序集的元数据信息. 当我们用Assembly类加载了一个程序集,我们可以搜索程序集中的类型信息. 它同样可以创建程序集中类型的实例.

## 动态加载一个程序集
Assembly类提供了以下方法在运行时加载一个程序集.

方法|说明
---|---
Load()|这个静态的可重载方法接收一个程序集名称参数,并在系统中搜索给定名称的程序集
LoadFrom()|这个静态的可重载方法接收一个完整的程序集文件路径, 它直接从指定的文件中加载程序集
GetExecutingAssembly()|得到当前运行的程序集信息.该方法没有重载.
GetTypes()|得到程序集中所有的类型
GetCustomAttributes()|这个静态可重载方法得到程序集中的所有自定义属性


    using System;
    using System.Reflection;

    namespace ReflectionDemo
    {
      internal class Program
      {
        private static void Main(string[] args)
        {
            Assembly objAssembly;

            // You must supply a valid fully qualified assembly name here.
            objAssembly = Assembly.Load("mscorlib,4.0.0.0,Neutral,b77a5c561934e089");

            //this loads currnly running process assembly
            // objAssembly = Assembly.GetExecutingAssembly();

            Type[] Types = objAssembly.GetTypes();

            // Display all the types contained in the specified assembly.
            foreach (Type objType in Types)
            {
                Console.WriteLine(objType.Name.ToString());
            }

            //fetching custom attributes within an assembly
            Attribute[] arrayAttributes =
            Attribute.GetCustomAttributes(objAssembly);

            // assembly1 is an Assembly object
            foreach (Attribute attrib in arrayAttributes)
            {
                Console.WriteLine(attrib.TypeId);
            }

            Console.ReadLine();
        }
      }
    }

该示例输入mscorlib程序集中所有的类型及自定义属性类.

## 延迟绑定
延迟绑定是.NET反射中的一个强大技术.它允许你在运行时创建给定类型的实例并调用它的成员. 这种技术也叫**动态调用**. 这种技术只有在一个对象在编译时不可用时有用. 使用这种技术时, 开发人员有责任保证调用方法的签名正确,否则, 会抛出异常. 相对于早期绑定, 编译器在方法调用前会验证方法的签名,并给出提示. 延迟绑定会影响性能,所有是否使用需要根据实际情况.

示例:

    using System;
    using System.Reflection;

    namespace ReflectionDemo
    {
      internal class Program
      {
        private static void Main(string[] args)
        {
            Assembly objAssembly;
            // Loads an assembly  
            objAssembly = Assembly.GetExecutingAssembly();

            //get the class type information in which late bindig applied 
            Type classType = objAssembly.GetType("ReflectionDemo.SportCar");

            //create the instance of class using System.Activator class 
            object obj = Activator.CreateInstance(classType);

            //get the method information 
            MethodInfo mi = classType.GetMethod("IsMoving");

            //Late Binding using Invoke method without parameters 
            bool isCarMoving;
            isCarMoving = (bool)mi.Invoke(obj, null);
            if (isCarMoving)
            {
                Console.WriteLine("Car Moving Status is : Moving");
            }
            else
            {
                Console.WriteLine("Car Moving Status is : Not Moving");
            }

            //Late Binding with parameters 
            object[] parameters = new object[3];
            parameters[0] = 32456;//parameter 1 startMiles 
            parameters[1] = 32810;//parameter 2 end Miles 
            parameters[2] = 10.6;//parameter 3 gallons 
            mi = classType.GetMethod("CalculateMPG");
            double MilesPerGallon;
            MilesPerGallon = (double)mi.Invoke(obj, parameters);
            Console.WriteLine("Miles per gallon is : " + MilesPerGallon);

            Console.ReadLine();
        }
      }
    }

输出:

    Car Moving Status is : Not Moving 
    Miles per gallon is : 33.3962264150943

## Reflection Emit
Reflection emit支持在运行时动态的创建新的类型. 你可以定义一个程序集去动态运行或保存到磁盘, 在程序集中你可以动态定义模块和模块中的类. 关于Reflection emit的更多信息,详见[MSDN](https://msdn.microsoft.com/en-us/library/system.reflection.emit(v=vs.110).aspx)和[CodeProject](http://www.codeproject.com/Articles/121568/Dynamic-Type-Using-Reflection-Emit)上的文章.

完.

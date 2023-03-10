# C# 9 方法和函数的新特性

> 原文：<https://developers.redhat.com/blog/2021/04/13/c-9-new-features-for-methods-and-functions>

这是我们 C# 9 系列的第三篇文章。在之前的文章中，我们介绍了[顶级程序和目标类型表达式](/blog/2021/03/30/c-9-top-level-programs-and-target-typed-expressions/)和[模式匹配的新特性](/blog/2021/04/06/c-9-pattern-matching/)。在本文中，我们将研究方法、匿名函数和局部函数的新特性。

## 协变返回类型

当重写基类成员或实现接口时，C# 9 允许您使用更具体的返回类型:

```
class Person
{
 public virtual Person Clone() { ... }
}

class Student : Person
{
 public override Student Clone() { ... }
}

```

在以前版本的 C#中，返回类型必须与基声明相匹配。为了获得更具体的实际类型，需要进行强制转换:

```
Student clone = (Student)student.Clone();

```

## 静态匿名函数

长期以来，C#支持使用匿名方法或 lambda 表达式声明匿名函数:

```
// Anonymous functions:
// - C# 2.0: anonymous methods
Func<string, int> = delegate(string arg) { return arg.Length; };
// - C# 3.0: lambda expressions
Func<string, int> = arg => arg.Length;

```

匿名函数可以使用局部变量。为了不允许这样做，并要求显式传递所有参数，我们现在可以将匿名函数标记为`static`:

```
// error CS8820: static anonymous function references ‘offset’
int offset = 20;
Func<string, int> d1 = static delegate(string arg) { return arg.Length + offset; };
Func<string, int> d2 = static arg => arg.Length + offset;

```

## 局部函数的属性

C# 7 引入了本地函数，它们是在调用方法中定义的。C# 8 增强了这些局部函数，并允许将它们标记为`static`以禁止使用局部变量(类似于上一节)。

C# 9 使得给局部函数添加属性成为可能。以下示例将`DllImport`属性应用于局部函数:

```
public static void Terminate(this Process p)
{
   const int SIGTERM = 15;
   kill(p.Id, SIGTERM);

   [DllImport("libc", SetLastError = true)]
   static extern int kill(int pid, int sig);
}

```

## 扩展分部方法

为了便于定制生成的代码，C# 3 引入了分部方法的概念。

生成的 C#代码包含一个标记为`partial`的方法，但没有主体。方法的主体由用户在单独的文件中提供。

C# 3 不要求用户提供主体。编译代码时，编译器使用用户提供的方法，如果没有这样的方法，则省略调用。

因为提供实现是可选的，所以分部方法不允许有输出参数或非 void 返回类型:

```
// -- MyForm.generated.cs --
public partial class MyForm : Form
{
   public MyForm()
   {
   	// ...
   	// generated code to initialize components.
   	// ...

   	OnComponentsInitialized();
   }

   partial void OnComponentsInitialized();
}

// -- MyForm.cs --
public partial class MyForm
{
   partial void OnComponentsInitialized()
   {
   	// user code
   }
}

```

C# 9 允许`partial`方法有一个返回类型和输出参数。编译器要求有一个实现。这种扩展的分部方法必须有一个可访问性修饰符，它不再局限于私有范围。

这允许用户将方法的声明(方法签名)与其定义(代码)分开。

扩展的分部方法与另一个新的 C#编译器特性密切相关:*源代码生成器*。源代码生成器是在编译过程中运行的代码，用于生成要编译的附加源代码。你可以在[介绍 C#源代码生成器](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/)中了解更多关于源代码生成器的知识。

正如您可能猜到的，源生成器负责生成分部方法的实现。以下示例显示了一个分部方法，该方法可以使源生成器发出一个针对在`RegexGenerated`属性中提供的正则表达式优化的实现:

```
[RegexGenerated("(dog]
public partial bool IsPetMatch(string input);

```

## 结论

本文介绍了 C# 9 中方法和函数的新功能。我们了解到被覆盖的方法现在可以返回更具体的类型，匿名方法可以被标记为`static`以要求传入所有参数，本地函数现在可以有属性。最后，我们看了扩展的分部方法以及它们如何与 C#源代码生成器一起使用。

在下一篇文章中，我们将看看 [`init`访问器和记录](/blog/2021/04/20/c-9-init-accessors-and-records/)。

C# 9 可以和[一起使用。NET 5 SDK](/blog/2020/12/22/net-5-0-now-available-for-red-hat-enterprise-linux-and-red-hat-openshift/) ，可以在[红帽企业 Linux](/products/rhel/overview) 和[红帽 OpenShift](/products/openshift/overview) 上，在 [Fedora](http://fedoraloves.net/) 和[上从微软获得，用于 Windows、macOS 和其他 Linux 发行版](https://dotnet.microsoft.com/download)。

*Last updated: October 14, 2022*
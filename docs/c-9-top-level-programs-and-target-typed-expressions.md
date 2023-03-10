# C# 9 顶级程序和目标类型表达式

> 原文：<https://developers.redhat.com/blog/2021/03/30/c-9-top-level-programs-and-target-typed-expressions>

[。NET](/topics/dotnet)5(2020 年 11 月发布)包括对 C# 9 的支持，c# 9 是 [C#](/topics/c) 编程语言的一个主要新版本。这一系列文章探索了。NET 的主要编程语言。在这第一篇文章中，我们将看看*顶级语句*和*目标类型的新的和条件表达式*。这些特性使得 C#不那么冗长，可以在日常程序中使用。

## 阅读整个系列

阅读本系列中介绍 C# 9 新特性的其他文章:

*   [第二部分:C# 9 模式匹配](/blog/2021/04/06/c-9-pattern-matching/)
*   第 3 部分:C# 9 方法和函数的新特性
*   [第 4 部分:C# 9 初始化访问器和记录](/blog/2021/04/20/c-9-init-accessors-and-records/)
*   [第 5 部分:更多的 C# 9](/blog/2021/04/27/some-more-c-9/)

此外，去年，我们发表了一系列关于 C# 8 的文章。您可以在这里找到这些文章:

*   [第 1 部分:C# 8 异步流](/blog/2020/02/24/c-8-asynchronous-streams/)
*   [第二部分:C# 8 模式匹配](/blog/2020/02/27/c-8-pattern-matching/)
*   [第三部分:C# 8 默认接口方法](/blog/2020/03/03/c-8-default-interface-methods/)
*   第 4 部分:C# 8 可空引用类型
*   [第五部分:更多的 C# 8](/blog/2020/03/11/some-more-c-8/)

## 顶级程序

顶级程序允许你编写应用程序的 main 方法，而不必用一个`static Main`方法添加一个`class`。例如:

```
// using directives
using static System.Console;
using System.Threading.Tasks;

// program statements
await Task.Delay(100);
WriteLine("Hello " + (args.Length > 0 ? args[0] : "world!"));
return 0;

// local functions 
// class/namespace declarations
void Foo() { }
class Foo { }

```

程序语句直接添加到 C#文件中，没有封闭的方法、类或命名空间。您可以将`using`指令放在程序语句之前。或者，在语句之后，可以定义局部函数、类型和命名空间。

这个例子还展示了顶级编程的一些有趣的特性。程序可以是异步的:我们可以使用`await`关键字。使用`args`参数可以得到程序参数，程序可能会返回一个退出代码。

## 目标类型的新表达式

从 C# 3 开始，我们可以使用`var`关键字省略变量的声明类型。编译器从表达式中派生类型:

```
var person = new Person();

```

使用 C# 9，您还可以从`new`操作符中省略类型，让编译器从声明类型中派生类型:

```
Person p1 = new();
Person p2 = new("Tom");
Person p3 = new() { FirstName = "Tom" };

```

这种语法的好处是类型声明很好地在左边对齐。正如您在示例中看到的，您可以传递构造函数参数并使用对象初始值设定项。

当您向方法传递参数时，目标类型的`new`表达式也可以工作。但是，不太清楚构造的是什么类型:

```
PrintPerson(new());

```

## 目标类型的条件表达式

使用 C# 9，`? .. : ..`表达式的分支可以有不同的类型，只要它们都转换成目标类型:

```
Control c = true ? button : form;

```

尽管 button 和 form 是不同的类型，但是这个例子在 C# 9 中仍然有效，因为两者都转换为目标类型(`Control`)。以前，分支需要具有完全相同的类型，这就要求在它们不匹配时引入强制转换。

## 结论

在本文中，我们研究了顶级程序，这些程序使得编写 main 方法不那么冗长。我们还讨论了目标类型的新表达式，它提供了一个很好的语法来对齐变量声明的类型，而不必为`new`操作符复制类型。最后，我们看到了当两个分支都转换为目标类型时，目标类型的条件表达式如何允许我们省略强制转换。

在下一篇文章中，我们将探索 C# 9 中[模式匹配的新特性。](https://developers.redhat.com/blog/2021/04/06/c-9-pattern-matching/)

您可以将 C# 9 与。NET 5 SDK，在[红帽企业版 Linux](/products/rhel/overview) 、[红帽 OpenShift](/products/openshift/overview) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 以及其他 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: April 28, 2021*
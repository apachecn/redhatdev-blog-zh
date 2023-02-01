# C# 8 默认接口方法

> 原文：<https://developers.redhat.com/blog/2020/03/03/c-8-default-interface-methods>

在之前的文章中，我们讨论了 C# 8 [异步流](https://developers.redhat.com/blog/2020/02/24/c-8-asynchronous-streams/)和[模式匹配](https://developers.redhat.com/blog/2020/02/27/c-8-pattern-matching/)。在这篇文章中，我们将看看 C# 8 的默认接口方法。

## 扩展接口

在 C# 8 之前，如果不破坏实现接口的类，就不可能向接口添加成员。因为接口成员是抽象的，所以类需要提供实现。C# 8 允许我们扩展一个接口并提供一个默认的实现。当类不提供默认实现时，运行时(也需要支持此功能)使用默认实现:

```
interface IOutput
{
    void PrintMessage(string message);
    void PrintException(Exception exception)
        => PrintMessage($"Exception: {exception}");
}
class ConsoleOutput : IOutput
{
    public void PrintMessage(string message)
        => Console.WriteLine(message);
}

```

在这个例子中，`ConsoleOutput`没有为`PrintException`提供实现。当对一个`ConsoleOutput`实例调用`PrintException`时，将调用来自`IOutput`接口的默认方法。`ConsoleOutput`可能会提供自己的实现。

通过显式实现基成员，派生接口可以提供更合适的默认实现:

```
interface IA
{
    void M() { WriteLine("IA.M"); }
}
interface IB : IA
{
    void IA.M() { WriteLine("IB.M"); }
}

```

我们通过在名称中包含`IA.`来显式实现基本成员。如果没有`IA.`，编译器会警告我们要么将其显式化，要么使用`new`关键字隐藏它。

C# 8 允许我们在接口中添加可以被默认接口成员使用的`static`成员:

```
interface IOutput
{
    private static string s_exceptionPrefix = "Exception";

    public static string ExceptionPrefix
    {
        get => s_exceptionPrefix;
        set => s_exceptionPrefix = value;
    }

    void PrintMessage(string message);

    sealed void PrintException(Exception exception)
        => PrintMessage($"{s_exceptionPrefix}: {exception}");
}

```

这个例子展示了一个`private static`字段和一个`public static`方法来实现由`sealed PrintException`方法使用的`ExceptionPrefix`属性。通过添加`sealed`关键字，这个方法不再能被覆盖。

## 代码继承

C#支持从单基类继承并实现多个接口。在 C# 8 之前，只有基类可以提供派生类可用的代码。使用 C# 8，接口可以向它们的实现类提供代码。

除此之外，我们可以在成员上使用访问修饰符，并提供静态成员:

```
interface IOutput
{
    sealed void PrintException(Exception exception)
        => PrintMessageCore($"Exception: {exception}");

    protected void PrintMessageCore(string message);

    protected static void PrintToConsole(string message)
        => Console.WriteLine(message);
}

class ConsoleOutput : IOutput
{
    void IOutput.PrintMessageCore(string message)
    {
        IOutput.PrintToConsole(message);
    }
}

```

在这个例子中，我们看到`IOutput`接口将`PrintMessageCore`实现委托给了派生类，并提供了一个利用了`PrintMessageCore`的`PrintException`的密封实现。这个例子还展示了如何从一个派生类型中调用`static protected PrintToConsole`方法。

这种设置允许我们包含接口的公共代码，允许不同的类在没有公共基类的情况下共享这些代码，从而支持[多重继承](https://en.wikipedia.org/wiki/Multiple_inheritance)和[特征](https://en.wikipedia.org/wiki/Trait_(computer_programming))模式。

C# 8 允许共享代码，但是接口仍然不允许有实例字段(状态)。需要状态的接口方法要么需要是抽象的(因此它们在类中实现)，要么需要接受状态作为参数(由调用者提供，就像实现类一样)。这种方法避免了状态的[继承菱形问题](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)。对于代码来说，菱形问题是通过编译器在不明确的情况下需要额外的成员来解决的(编译器随后可以调用适当的基)。

# 结论

在本文中，我们研究了 C# 8 默认接口方法。默认接口方法提供了一种用新成员扩展接口的方式，而不会破坏以前的实现者。新的特性还允许代码在不同类型之间共享，这使得[多重继承](https://en.wikipedia.org/wiki/Multiple_inheritance)和[特征](https://en.wikipedia.org/wiki/Trait_(computer_programming))模式成为可能。在下一篇文章中，我们将关注[可空引用类型](https://developers.redhat.com/blog/2020/03/05/c-8-nullable-reference-types/)。

C# 8 可以与。NET Core 3.1 SDK，在[红帽企业版 Linux](https://access.redhat.com/documentation/en-us/net_core/) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 等 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: December 4, 2020*
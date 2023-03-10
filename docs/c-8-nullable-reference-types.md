# C# 8 可空引用类型

> 原文：<https://developers.redhat.com/blog/2020/03/05/c-8-nullable-reference-types>

在上一篇文章中，我们讨论了 [C# 8 的默认接口方法](https://developers.redhat.com/blog/2020/03/03/c-8-default-interface-methods/)。在本文中，我们将研究 C# 8 可空引用类型。引用类型是指堆上的对象。当没有对象引用时，值为`null`。有时`null`是一个可接受的值，但通常它是一个非法值，导致了`ArgumentNullExceptions`和`NullReferenceExceptions`。

C# 8 终于给了我们表达一个变量是否不应该是`null`，什么时候可以是`null`的能力。基于这些注释，当你潜在地使用一个`null`引用，或者传递一个`null`引用给一个不接受它的函数时，编译器会警告你。

要启用该功能，您必须在项目文件(`csproj` ) `PropertyGroup`中添加以下行:

```
<Nullable>enable</Nullable>

```

除了`enable`之外的其他值也是可能的，这使得编译器更加宽松。也可以更改每个文件的设置。更多信息参见[可空上下文](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references#nullable-contexts)文档。

`?`字符用于指示一个类型何时可能是`null`。当`?`字符不存在时，类型被认为是不可空的:

```
MyClass? mayBeNull = null;
MyClass  mayNotBeNull = new MyClass();

```

我们也可以对方法参数和返回值使用以下语法:

```
static MyClass Foo(MyClass? mayBeNull)
{
    return mayBeNull ?? new MyClass();
}

```

方法`Foo`接受一个可能是`null`的参数，并返回一个不是的参数。

编译器不仅仅依赖于类型声明。它还遵循代码路径来理解一个类型何时不再是`null`:

```
static void Bar(MyClass? arg)
{
    if (arg == null)
    {
        throw new ArgumentNullException(nameof(arg));
    }

    arg.Method();
}

```

在这个例子中，编译器知道当我们调用`arg.Method`时`arg`不能是`null`,因为我们之前已经抛出了。如果我们忽略了`null`检查，那么我们会得到一个关于`arg.Method`调用的警告:

`CS8602: Dereference of a possibly null reference`。

假设我们不小心错过了键入关键字`throw`。在这种情况下，我们也会得到警告。

可能会有这样的情况，编译器不知道一个变量不再是`null`。例如，如果您的类只允许在您确定已经设置了字段的状态下调用一个方法。我们可以使用`null`-宽容操作符(`!`)将这些信息传递给编译器:

```
string scheme = _uri!.Scheme; // _uri is of type Uri?
int port = _uri.Port;

```

因为我们在变量上使用了`!`操作符，编译器知道它是安全的。当我们访问`Port`时，我们不需要在下一行使用`null`-宽容操作符。编译器推断`_uri`不再是`null`。

添加到方法参数中的可空注释作为`Attributes`成为程序集的一部分。。NET Core 基本库正在更新以包含这些注释。

编译器在方法中检查。当该方法对于可空参数的行为有很强的保证时，该方法可以用表达这一点的`Attributes`来扩充:

```
static void Bar(MyClass? arg)
{
    if (arg == null)
    {
        ThrowArgumentNull(nameof(arg));
    }

    arg.Method();
}

[DoesNotReturn]
public static void ThrowArgumentNull(string paramName)
{
    throw new ArgumentNullException(paramName);
}

```

在这个简单的例子中，我们使用了`DoesNotReturn`属性，它告诉编译器`arg`不会通过`if`块，因为`ThrowArgumentNull`永远不会返回。一些属性可以以值为条件。例如，`string.IsNullOrEmpty`注释如下:

```
public static bool IsNullOrEmpty([NotNullWhenAttribute(false)] string? value)

```

这个注释让编译器明白，当这个方法返回`false`时，`value`参数不是`null`。

您可以在[系统中找到这些属性以及更多属性。代码分析](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis?view=netcore-3.1)命名空间。这些属性在中可用。网芯 3.x(以及`netstandard2.1`)。

# 结论

C# 8 为我们提供了一种机制来表达什么时候引用类型可能是`null`而什么时候不是。多亏了这些注释，我们现在可以表达 API 的可空性预期。编译器使用这些信息来提供警告，否则可能会在运行时导致意外异常。更多新特性，请看我们系列的最后一部分， [*再来点 C# 8*](https://developers.redhat.com/blog/2020/03/11/some-more-c-8/) 。

C# 8 可以与。NET Core 3.1 SDK，在 [RHEL](https://access.redhat.com/documentation/en-us/net_core/) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 以及其他 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: December 10, 2020*
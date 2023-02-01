# 再来点 C# 9

> 原文：<https://developers.redhat.com/blog/2021/04/27/some-more-c-9>

在 C# 9 系列的最后一篇文章中，我们将看看 C# 9 中与[本机互操作](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interop/interoperability-overview)和性能相关的高级特性。

如果您错过了本系列中的任何一篇文章，您可以在这里补上:

*   第一部分:C# 9 顶级程序和目标类型表达式
*   [第二部分:C# 9 模式匹配](/blog/2021/04/06/c-9-pattern-matching/)
*   第 3 部分:C# 9 方法和函数的新特性
*   [第 4 部分:C# 9 初始化访问器和记录](/blog/2021/04/20/c-9-init-accessors-and-records/)

## 本机大小的整数

C# 9 引入了对本地整数类型的语言支持，包括有符号和无符号。现有的 C# `int`和`long`类型映射到底层的`System.Int32`和`System.Int64`类型，分别具有 32 位和 64 位的固定大小。新的`nint`和`nuint`类型映射到现有的`System.IntPtr`和`System.UIntPtr`类型，其尺寸对应于本机尺寸。这意味着它们在 32 位机器上是 32 位，在 64 位机器上是 64 位。

C#支持对`nint`和`nuint`的直接运算:

```
// IntPtr aren't directly usable for arithmetic.
IntPtr a = (IntPtr)5;
IntPtr b = (IntPtr)6;
IntPtr c = new IntPtr((long)a + (long)b);

// nint/nuint can be used for arithmetic.
nint i = 5;
nint j = 6;
nint k = i + j;

// cast required: int-size may be smaller.
int y = (int)i;

```

原生整数可以标记为`const`，但是只有当编译器知道结果不会在任何架构上溢出时:

```
const nint n = int.MaxValue;
// error CS0133: The expression being assigned to 'm' must be constant.
const nint m = unchecked(n + 1);

```

本地库经常在它们的 API 中使用机器大小的本地整数，新的`nint` / `nuint`有助于从。网络:

```
public unsafe int write(int fd, Span buffer)
{
 fixed(byte* buf = buffer)
 {
   return (int)write(fd, buf, (nuint)buffer.Length);
 }
 // ssize_t write(int fd, const void *buf, size_t count);
 //   ssize_t, and size_t are signed/unsigned native-size integer types.
 [DllImport("libc", SetLastError = true)]
 static extern nint write(int fd, void* buf, nuint count);
}

```

## 抑制 localsinit

`localsinit`功能可以提高性能。要理解它，请看下面的方法:

```
void ReadData()
{
   Span buffer = stackalloc byte[256];
   int bytesRead = Read(buffer);
   buffer = buffer.Slice(0, bytesRead);
   ProcessData(buffer);
}

```

每次调用该方法时，运行时都会确保分配的缓冲区被归零，这样该方法就不会从堆栈中读取未初始化的数据。C#编译器在方法中发出`.locals init`指令来实现这一点。

当性能至关重要时，可能需要省略这种零位调整。C# 9 通过新的 [`SkipLocalsInit`](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.skiplocalsinitattribute) 属性支持性能度量。此属性可以应用于方法级别、类级别，甚至整个模块:

```
// For the entire project.
[module: System.Runtime.CompilerServices.SkipLocalsInit]

// or, for a  specific method.
[SkipLocalsInit]
void ReadData()
{
   // ...

```

如果您添加这个属性，请注意初始化用作`out`参数的变量，或者您将其地址传递给另一个函数的变量。这些变量将不再被编译器归零:

```
[SkipLocalsInit]
unsafe void Foo()
{
   int i; // i is not cleared.
   Bar(&i);
}

```

## 模块初始化器

C#支持`static`类型构造函数，当类型第一次被使用时，它只被调用一次。C# 9 使得在汇编级添加构造函数成为可能(更具体的是:[模块级](https://docs.microsoft.com/en-us/archive/blogs/junfeng/netmodule-vs-assembly))。这个模块初始化器在模块第一次使用时运行一次。

与静态构造函数相比，模块初始化器更有效，因为运行时不必跟踪是否为每种类型调用了构造函数。此外，模块初始化器避免了可能存在于多个静态构造函数之间的排序问题。

通过添加 [ModuleInitializer](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.moduleinitializerattribute) 属性，可以将一个方法标记为模块初始化器。该方法必须是`public`或`internal`，具有`void`返回类型，并且不得接受参数:

```
static void Main(string[] args)
{
   Console.WriteLine("Hello from Main.");
}

[ModuleInitializer]
static internal void MyInitializer()
{
    Console.WriteLine("Hello from initializer!");
}

```

多个方法可以有`ModuleInitializer`属性。编译器生成一个模块初始化器，调用所有的初始化器。

## 函数指针

C#一直支持使用*委托*的传递方法。委托是引用类型，引用一个或多个可以通过委托调用的方法:

```
Action<string> myDelegate = s => Console.WriteLine($"Hello 1 {s}");
myDelegate += s => Console.WriteLine($"Hello 2 {s}");
myDelegate("world!");

```

C# 9 更进一步，允许你在 C#中直接使用函数指针。函数指针只包含函数的地址。它们只能在不安全的块中使用。

函数指针可以引用托管静态方法或本机函数。对于本机函数，如果缺省值不合适，可以进一步指定调用约定。这告诉 JIT 需要如何将参数传递给函数。

使用`delegate*`声明函数指针。下一个示例声明了三个函数指针:

```
var pFun1 = (delegate* unmanaged<int, void>)NativeLibrary.GetExport(libHandle, "fun1");

var pFun2 = (delegate* unmanaged[Stdcall]<int, void>)NativeLibrary.GetExport(libHandle, "fun2");

delegate*<int, void> pManagedFunction = &ManagedFunction;

```

前两个指针是`unmanaged`，使用[本地库](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.nativelibrary)类初始化。类型参数与`Func`委托的类型参数相同:首先是参数类型，最后是返回类型。这里声明的指针接受一个`int`，没有返回类型(`void`)。

第二个函数指针指定了 [Stdcall](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callconvstdcall) 调用约定，以改变平台默认值。

第三个函数指针是托管函数指针。它是使用静态方法上的 address-of 运算符来赋值的。

可以使用预期的调用语法进行函数调用:

```
pManagedFunction(10);

```

一种特殊情况是将托管方法的地址传递给本机库。为此，我们需要向该方法添加 [UnmanagedCallersOnly](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.unmanagedcallersonlyattribute) 属性。这告诉运行时将从本机代码中调用该方法。此类方法不得从托管代码中调用，并且只能有[可直接复制到本机结构中的](https://docs.microsoft.com/en-us/dotnet/framework/interop/blittable-and-non-blittable-types)参数:

```
[UnmanagedCallersOnly]
static void Log(IntPtr ptr)
{
  Console.Write(Marshal.PtrToStringAnsi(ptr));
}

static unsafe void Main(string[] args)
{
  delegate* unmanaged<IntPtr, void> pLog = &Log;
  nativelib_set_log_function(pLog);
  ...
}

```

## 结论

在本文中，我们研究了新的`nint`和`nuint`原生整数类型，以及它们如何促进互操作。我们了解了`SkipLocalsInit`属性如何避免将堆栈局部变量清零的开销。我们讨论了当一个程序集或模块第一次被使用时，模块初始化器如何允许你运行一个或多个方法。最后，我们看了函数指针以及它们如何用于托管和非托管函数。

C# 9 可以和[一起使用。NET 5 SDK](/blog/2020/12/22/net-5-0-now-available-for-red-hat-enterprise-linux-and-red-hat-openshift/) ，在[红帽企业版 Linux](/products/rhel/overview) 、[红帽 OpenShift](/products/openshift/overview) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 等 Linux 发行版](https://dotnet.microsoft.com/download)上都有。有关 Red Hat 对。NET，见红帽开发者[。NET 话题页面](/topics/dotnet)。

*Last updated: April 26, 2021*
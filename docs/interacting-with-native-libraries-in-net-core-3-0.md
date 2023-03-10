# 在中与本地库交互。网络核心 3.0

> 原文：<https://developers.redhat.com/blog/2019/09/06/interacting-with-native-libraries-in-net-core-3-0>

`NativeLibrary`是中的一个新类。NET Core 3.0，用于与本地库交互。在本文中，我们将仔细研究一下。

## dl lim port(dl lim port)-dl lim port(dl lim port)-dl lim port(dl lim port)(dl lim port)(dl lim port)(dl lim port)(dl lim port)(dl lim port)(dl lim port)(dl lim port)(dl lim port

。NET 使得使用`DllImport`从本地库中调用函数变得简单:

```
[DllImport("mylibrary")]
public static extern int foo();

```

这段代码使得来自本地库`mylibrary`的函数`foo`可用。该函数不接受任何参数，并返回一个`int`。。NET 负责封送参数类型。可以使用自动封送的托管类型(如字符串)。

当我们使用这个函数时。网芯试着找`mylibrary`。它在应用程序文件夹和系统库文件夹中查找。当查找时，它会尝试名称的变体。比如在 Windows 上，它增加了一个`.dll`扩展名；在 Linux 上，它添加了一个`.so`扩展名。该查找还考虑了基于[运行时标识符](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog) (RID)的当前平台。一个应用程序可以包含不同运行时标识符的库(组织在 rid-folders 中)，并且将使用最合适的库。

`DllImport`的主要限制是库名和符号名在编译时是固定的。在许多情况下，特别是当您自己构建库并将其包含在应用程序中时，这种限制不是问题。

## 本地图书馆

`NativeLibrary`是一个只有几个方法的静态类:

```
void Free(IntPtr handle)
IntPtr GetExport(IntPtr handle, String name)
IntPtr Load(String libraryPath)
IntPtr Load(String libraryName, Assembly, DllImportSearchPath?)
void SetDllImportResolver(Assembly, DllImportResolver)
IntPtr TryGetExport(IntPtr handle, String name, out IntPtr address)
IntPtr TryLoad(String libraryPath, out IntPtr handle)
IntPtr TryLoad(String libraryName, Assembly, DllImportSearchPath?, out IntPtr handle)

```

我们可以做的第一件事是通过为我们的程序集提供一个`DllImportResolver`委托来控制我们在`DllImport`中使用的库。`DllImportResolver`有如下签名:

```
public delegate IntPtr DllImportResolver(string libraryName, Assembly assembly, DllImportSearchPath? searchPath);

```

它的参数为我们提供了`DllImport`的上下文，作为返回值，我们必须为库提供一个`IntPtr`。我们使用`Load`方法得到这个`IntPtr`。`Load(string)`方法在一个特定的路径加载库。另一个`Load`方法提供默认的`DllImport`加载逻辑。让我们看看如何使用这些信息:

```
static class Library
{
const string MyLibrary = "mylibrary";

static Library()
{
    NativeLibrary.SetDllImportResolver(typeof(Library).Assembly, ImportResolver);
}

private static IntPtr ImportResolver(string libraryName, Assembly assembly, DllImportSearchPath? searchPath)
{
    IntPtr libHandle = IntPtr.Zero;
    if (libraryName == MyLibrary)
    {
        // Try using the system library 'libmylibrary.so.5'
        NativeLibrary.TryLoad("libmylibrary.so.5", assembly, DllImportSearchPath.System32, out libHandle);
    }
    return libHandle;
}

[DllImport(MyLibrary)]
public static extern int foo();
}

```

在这个例子中，我们为程序集注册了一个`DllImportResolver`。在我们的解析器中，我们尝试从系统库中加载`libmylibrary.so.5`。如果失败，我们通过返回`IntPtr.Zero`返回默认的`DllImport`分辨率。这个结果给了我们`DllImport`的可用性，以及在运行时选择特定库的灵活性。

我们可以使用`NativeLibrary`做的另一件事是使用`GetExport` / `TryGetExport`直接解析符号。让我们看一个例子:

```
class Library : IDisposable
{
private readonly IntPtr _libHandle;
private readonly Func<int> _foo;
private bool _disposed;

public Library()
{
    _libHandle = NativeLibrary.Load("mylibrary", typeof(Library).Assembly, DllImportSearchPath.System32);

    if (NativeLibrary.TryGetExport(_libHandle, "foo", out IntPtr fooHandle))
    {
        _foo = Marshal.GetDelegateForFunctionPointer<Func<int>>(fooHandle);
    }
    else
    {
        _foo = () => { throw new NotSupportedException("'foo' not found"); };
    }
}

~Library()
{
    Dispose(false);
}

public int foo()
{
    ThrowIfDisposed();
    return _foo();
}

public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}

protected virtual void Dispose(bool disposing)
{
    if (!_disposed)
    {
        _disposed = true;
        NativeLibrary.Free(_libHandle);
    }
}

private void ThrowIfDisposed()
{
    if (_disposed)
    {
        ThrowObjectDisposedException();
    }
}

private void ThrowObjectDisposedException()
    => throw new ObjectDisposedException(typeof(Library).FullName);
}

```

这里我们用对`NativeLibrary.Load`、`TryGetExport`和`Marshal.GetDelegateForFunctionPointer`的调用替换了`DllImport`。这涉及到更多的代码，但是作为回报，我们现在可以完全控制我们正在使用的库，并且可以动态地检测和使用它的符号。

## 结论

在本文中，您了解了新的`NativeLibrary`类，以及当您需要更多地控制库分辨率和所使用的符号时，如何使用它来代替`DllImport`属性。
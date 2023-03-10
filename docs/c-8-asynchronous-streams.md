# C# 8 异步流

> 原文：<https://developers.redhat.com/blog/2020/02/24/c-8-asynchronous-streams>

。NET Core 3.1(2019 年 12 月)包括对 C# 8 的支持，这是 C#编程语言的一个新的主要版本。在这一系列文章中，我们将研究。NET 的主要编程语言。第一篇文章特别关注异步流。这个特性使得创建和使用异步枚举变得很容易，所以在开始使用这个新特性之前，首先需要理解 IEnumerable 接口。

**注意:** C# 8 可以与。NET Core 3.1 SDK，可在[红帽企业版 Linux](https://access.redhat.com/documentation/en-us/net_core/) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 以及其他 Linux 发行版](https://dotnet.microsoft.com/download)上获得。

## 阅读整个系列

阅读本系列中介绍。NET 的主要编程语言:

*   [第 1 部分:C# 8 异步流](https://developers.redhat.com/blog/2020/02/24/c-8-asynchronous-streams/)
*   [第二部分:C# 8 模式匹配](https://developers.redhat.com/blog/2020/02/27/c-8-pattern-matching/)
*   [第三部分:C# 8 默认接口方法](https://developers.redhat.com/blog/2020/03/03/c-8-default-interface-methods/)
*   第 4 部分:C# 8 可空引用类型
*   [第五部分:更多的 C# 8](https://developers.redhat.com/blog/2020/03/11/some-more-c-8/)

## IEnumerable 简史

经典的`IEnumerable<T>`从那以后就一直存在。NET 框架 2 (2005)。这个接口为我们提供了一种类型安全的方法来迭代任何集合。

迭代基于`IEnumerator<T>`类型:

```
public interface IEnumerator<T> : IDisposable
{
    bool MoveNext();
    T Current;
    void Reset();
}

```

您可以看到,`MoveNext`方法将我们移动到下一个元素。当*是*元素时，它返回`true`，然后`Current`返回该元素。`Reset`方法提供了一种将迭代器重置到起点的方法。`IEnumerable<T>`是`IDisposable`，因此它的实现可能会执行资源清理。注意，通用参数是一个`out`参数。该关键字允许将`IEnumerable<T>`转换为基本类型`IEnumerable<TBase>`。

`foreach`关键字允许轻松消费`IEnumerables`。

```
foreach (var item in myEnumerable)
   Console.WriteLine($”* {item}”);

```

`yield`关键字使得用一个方法实现一个`IEnumerable<T>`变得容易，并且让编译器弄清楚如何实现这个接口。例如:

```
IEnumerable<int> MyEnumerable
{
    get
    {
        for (int i = 0; i < 3; i++)
            yield return i;
        yield return 100;
    }
}

```

这段代码使编译器生成一个实现`IEnumerable<int>`的类型，它跟踪足够的信息来知道我们在`[0, 1, 2, 100]`迭代中的位置。

C# 3 的语言集成查询(2007)让`IEnumerable<T>`大放异彩，它允许`IEnumerable<T>`被转换、组合和过滤。例如:

```
var queryLondonCustomers = from cust in customers
                           where cust.City == "London"
                           select cust.Name;

```

本例中的`customers`指的是一个`IEnumerable<Customer>`。我们按城市过滤，这(再次)给了我们一个`IEnumerable<Customer>`。最后，我们选择名称，这样我们得到的类型就是一个`IEnumerable<string>`。

## 异步流

异步流使用`IAsyncEnumerator<T>`类型。这个类型类似于`IEnumerator<T>`，但是有一个异步的`Move`方法(它返回一个类似`Task`的类型):

```
ValueTask<T> MoveNextAsync();

```

多亏了 async `Move`方法，我们现在可以异步等待下一项。这意味着我们可以在不阻塞线程的情况下等待。注意，该方法返回一个`ValueTask`，当下一个项目已经可用时，它使调用免于分配。你可以在[了解价值任务](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/)的原因、内容和时间中阅读更多关于`ValueTask`的内容。

`IAsyncEnumerable<T>`非常适合不经常发生的事件，或者异步接收的数据(例如，通过网络)。因为`IAsyncEnumerable`知道我们拉取项目的速率，所以它可以智能地知道它缓冲了多少数据，以及它何时向上游源请求更多数据。

和`IEnumerable<T>`一样，C#为实现`IAsyncEnumerables`提供了一流的支持。例如:

```
static async IAsyncEnumerable<string> GetTopSearchResults(string term)
{
    using var client = new HttpClient();
    yield return await client.GetStringAsync($"https://www.google.com?q={term}");
    yield return await client.GetStringAsync($"https://www.bing.com?q={term}");
}

```

对于消费，我们使用`await foreach`关键字，如您在本例中所见:

```
await foreach (var item in GetTopSearchResults("test"))
{
    System.Console.WriteLine(item);
}

```

# 取消

取消异步方法的默认模式是使用`CancellationToken`。我们可以像这样给我们的方法添加一个`CancellationToken`:

```
static async IAsyncEnumerable GetTopSearchResults(string term, [EnumeratorCancellation]CancellationToken ct = default)
{
    using var client = new HttpClient();
    // GetStringAsync doesn't accept a CancellationToken, Dispose the client to cancel.
    using var ctr = ct.Register(s => ((HttpClient)s).Dispose(), client);
    yield return await client.GetStringAsync($"https://www.google.com?q={term}");
    yield return await client.GetStringAsync($"https://www.bing.com?q={term}");
}

```

当我们调用方法时，我们可以将`CancellationToken`作为参数传递:

```
var cts = new CancellationTokenSource(millisecondsDelay: 1000);
await foreach (var result in GetTopSearchResults("dotnet", cts.Token))

```

或者，我们可以使用`WithCancellation`方法，让编译器将值传递给带有`EnumeratorCancellation`属性的参数。例如:

```
var cts = new CancellationTokenSource(millisecondsDelay: 1000);
await foreach (var result in GetTopSearchResults("dotnet").WithCancellation(cts.Token))

```

## `ConfigureAwait(false)`

当异步方法异步完成时，它使用调用`SynchronizationContext`继续。当在 Win32 窗体/WPF 应用程序中调用异步方法时，此功能使我们回到主 UI 线程。它简化了 GUI 编程，因为 UI 控件应该只从 UI 线程更新。当不需要这个特性时，可以通过向等待的任务添加`ConfigureAwait(false)`来避免开销，如下所示:

```
await SomeMethodAsync().ConfigureAwait(false);

```

我们可以将`ConfigureAwait(false)`应用于由`foreach`语句生成的`MoveNextAsync`调用，如下所示:

```
await foreach (var result in GetTopSearchResults("dotnet").ConfigureAwait(false))

```

请注意，此选项适用于`MoveNextAsync`调用。您还应该将`ConfigureAwait(false)`添加到`GetTopSearchResults`方法中等待的方法中。

# 结论

在本文中，您了解了 C# 8 异步流如何使创建和使用异步可枚举数变得容易。异步流是一个有趣的特性，特别是对于使用从网络接收的数据/事件的应用程序。在本系列的下一篇文章中，我们将看看 [C# 8 的扩展模式匹配](https://developers.redhat.com/blog/2020/02/27/c-8-pattern-matching/)。

*Last updated: December 23, 2020*
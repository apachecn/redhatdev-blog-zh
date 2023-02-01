# 再来点 C# 8

> 原文：<https://developers.redhat.com/blog/2020/03/11/some-more-c-8>

在之前的文章中，我们介绍了 [C# 8 异步流](https://developers.redhat.com/blog/2020/02/24/c-8-asynchronous-streams/)、 [C# 8 模式匹配](https://developers.redhat.com/blog/2020/02/27/c-8-pattern-matching/)、 [C# 8 默认接口方法](https://developers.redhat.com/blog/2020/03/03/c-8-default-interface-methods/)和 [C# 8 可空引用类型](https://developers.redhat.com/blog/2020/03/05/c-8-nullable-reference-types/)。在这最后一篇文章中，我们将看看`static`局部函数、索引和范围，以及`using`声明。

## `static`本地功能

C# 7 引入了本地函数，这些函数在调用函数中定义和使用。这样的函数可以改变它们的调用者的局部变量。为了禁止这种可能性并要求显式传递所有参数，现在可以将局部函数标记为`static`:

```
void Bar()
{
  int i = 0;
  Foo();

  static void Foo()
  {
    i = 3; // CS8421: A static local function cannot contain a reference to 'i'
  }
}

```

## 指数和范围

C# 8 增加了一个范围的语法。该语法由范围运算符(`..`)组成，该运算符由一个*开始*和*结束*表达式包围，该表达式指定第一个元素(包括在内)的索引和最后一个元素(排除在外)的索引:

```
string[] fruits = new string[]
{
    "apple",
    "banana",
    "cherry",
};

string[] allFruits = fruits[0..3];
string[] allFruits2 = fruits[0..fruits.Length];
string[] allFruits3 = fruits[0..(2 + 1)];

```

在上面的例子中，我们通过将`0`指定为起始索引，将`3`指定为结束索引来获取所有水果的范围。结束索引是最后一个元素之后的一个，因为 C#索引是从零开始的。因为结束索引是*而不是*包含在范围内，所以我们增加了一个。

也可以使用`^`操作符从末尾开始索引。`^0`是最后一个元素之后的索引，`^1`是最后一个元素，`^2`是它之前的元素，依此类推。

这里有一个例子:

```
string[] lastTwoFruits = fruits[^2..^0];
```

默认情况下，开始索引是`0`，结束索引是`^0`，因此可以省略其中一个或两个:

```
string[] allFruits4 = fruits[..];
string[] skipFirstTwoFruits = fruits[2..];

```

反向索引也可以直接使用:

```
string lastFruit = fruits[^1];

```

C#编译器使用`System.Index`和`System.Range`类型来表示范围和索引。您可以通过添加接受这些类型的[索引器](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/indexers/)来支持您自己的类型。已经有一个采用`int`和`int Count/Length`属性的索引器的类型隐式支持反向索引。额外包含一个采用两个`ints`(假定偏移量和计数)的`Slice`方法的类型隐式支持采用一个范围。

的。NET 数组、`string`和`Span`支持索引和范围。`List`类型支持索引，但不支持范围。

## `using`声明

C#支持用关键字`using`处理变量。以前版本的 C#需要一个显式限定生存期范围的 block 语句:

```
using (FileStream fs = File.OpenRead("myfile.txt"))
{
    // explicit block
    ReadFromStream(fs);
}

```

有了 C# 8，就不再需要这个块了。编译器将使用声明范围:

```
using FileStream fs = File.Open("myfile.txt");
ReadFromStream(fs);

```

如你所见，这减少了缩进。

# 结论

在本文中，我们研究了三种不同的 C# 8 特性:

*   `static`局部函数，要求将所有参数显式传递给局部函数。
*   索引和范围，引入一流的支持来表示范围和执行反向索引。
*   `using`声明，在处理一次性对象时减少缩进量。

C# 8 可以与。NET Core 3.1 SDK，在 [RHEL](https://access.redhat.com/documentation/en-us/net_core/) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 以及其他 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: June 29, 2020*
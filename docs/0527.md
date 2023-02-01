# C# 8 模式匹配

> 原文：<https://developers.redhat.com/blog/2020/02/27/c-8-pattern-matching>

在上一篇文章中，我们研究了 [C# 8 异步流](https://developers.redhat.com/blog/2020/02/24/c-8-asynchronous-streams/)。C# 8 的另一个新特性是对模式匹配的扩展支持。在这篇文章中，我们将看看 C# 7 有哪些功能，C# 8 增加了哪些功能。

## C# 7 模式匹配

模式匹配是 C# 7 中引入的一个特性。它允许您通过使用`is`模式和`case`模式，以简洁的方式检查对象是否属于特定类型并检查其值。

### `is`模式

`is`模式允许你检查一个变量是否属于某种类型，然后把它赋给一个新的变量。然后可以对该变量进行进一步检查:

```
if (input is int count && count > 100)

```

该模式也可用于检查变量是否为`null`:

```
if (input is null)

```

第二条语句保证会进行`null`引用检查。当您使用`== null`时，`operator==`可能会过载，导致执行不同的检查。

### `case`模式

`switch`语句案例也支持模式。这些模式可以包括类型检查，以及附加条件:

```
switch (i)
{
    case int n when n > 100:
      ...
    case Car c:
      ...
    case null:
      ...
    case var j when (j.Equals(10)):
      ...
    default:
      ...
}

```

在上面的例子中，你可以看到`null`、`default`、类型检查、条件和没有类型检查的使用条件(`case var`)的`case`语句。注意，`case var`也可以匹配`null`，所以为了避免这种情况发生，我们把它放在了`case null`下面。

经典的`switch`语句只允许常量。由于动态条件，模式案例的顺序很重要。

## C# 8 模式匹配

C# 8 扩展了对模式的支持以及它们的使用范围。

### `switch`表情

`switch`表达式是一种基于另一个值返回特定值的简洁方法:

```
var rgbColor = knownColor switch
{
    KnownColor.Red   => new RGBColor(0xFF, 0x00, 0x00),
    KnownColor.Green => new RGBColor(0x00, 0xFF, 0x00),
    ...
    _                => throw new ArgumentException(message: "invalid enum value", paramName: nameof(knownColor)),
};

```

常规的`switch`不返回值。这种语法更简洁。没有`case`关键词，`default`案被替换成了一个丢弃(`_`)。

`case`条件可以是模式。不可能包含处理案件的语句。对于每种情况，必须提供一个表示结果值的表达式。这个表达式可以是一个`switch`表达式。

也可以从多个输入值开始，将它们收集到一个元组中:

```
public decimal GetDiscount(CustomerType customer, DiscountPeriod period)  =>
    (customer, period) switch
    {
        (CustomerType.Gold,   DiscountPeriod.Christmas)  => 0.2m,
        (CustomerType.Silver, DiscountPeriod.Christmas)  => 0.1m,
        (_,                   DiscountPeriod.Christmas)  => 0.05m,
        (_, _)                                           => 0m,
    };

```

`switch`的条件现在也是元组。它们的项目具有与相应的输入元组元素相匹配的模式。在这个例子中，我们使用了 discard ( `_`)来忽略一个元组项。也可以使用其他模式。

`is`关键字也可以用于元组模式。

元组模式也可以用于那些[可以解构为元组](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7#tuples)的类型:

```
static Sector GetSector(Point point)  => point switch
{
    (0, 0) => Sector.Origin,
    (2, _) => Sector.One,
    var (x, y) when x > 0 && y > 0 => Sector.Two,
    (1, var y) when y < 0 => Sector.Three,
    _ => Sector.Unknown
};

```

### 属性模式

属性模式表示需要有特定常数值的属性:

```
switch (location)
{
   case { State: "MN" }:
      ...
}

```

当`location.State`等于`MN`时，上述`case`将匹配。属性模式也可以用在开关表达式中。

一个特例是`{ }`模式，意思是:不是`null`。这种模式也可以和`is`关键字一起使用:

```
if (location is { State: "MN" })

```

我们可以检查类型和属性，例如:

```
switch (vehicle)
{
    case Taxi { Occupants: 2 } t:
      ...
}

```

## 结论

在本文中，我们已经了解了 C#对模式匹配的支持。模式匹配为我们提供了针对某个类型的简明语法匹配，检查属性，并将这些模式与附加条件相结合。在下一篇文章中，我们将探索对 C# 8 的默认接口方法的增强。

C# 8 可以与。NET Core 3.1 SDK，在[红帽企业版 Linux](https://access.redhat.com/documentation/en-us/net_core/) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 等 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: June 29, 2020*
# C# 9 初始化访问器和记录

> 原文：<https://developers.redhat.com/blog/2021/04/20/c-9-init-accessors-and-records>

这是我们 C# 9 系列的第四篇文章。之前的文章涵盖了[顶级程序和目标类型表达式](https://developers.redhat.com/blog/2021/03/30/c-9-top-level-programs-and-target-typed-expressions/)、[模式匹配的新特性](https://developers.redhat.com/blog/2021/04/06/c-9-pattern-matching/)和[方法和函数的新特性。](https://developers.redhat.com/blog/2021/04/13/c-9-new-features-for-methods-and-functions/)

在本文中，我们将看看`init`访问器和记录。

## 初始化访问器

C# 9 为名为`init`的属性引入了一个新的访问器。`init`访问器的工作方式类似于`public set`访问器，但是它只能在对象初始化期间使用。尝试在其他地方设置属性会产生编译错误:

```
public class Person
{
 public string FirstName { get; init; }
 public string LastName { get; init; }
}

Person person = new()
{
  FirstName = "John"       // OK.
};

person.FirstName = "Jane"; // error CS8852.

```

`init`访问器可以在构造函数中使用。并且，可以为访问器提供一个主体，它可以更改`readonly`字段:

```
public class Person
{
 private readonly string _firstName;
 public Person(string firstName)
 {
   FirstName = firstName;
 }
 public string FirstName
 {
   get => _firstName;
   init =>
 	_firstName = value ?? throw new ArgumentNullException(nameof(value));
 }
 public string LastName { get; init; }
}

```

在构造函数之后将调用`init`访问器(在对象初始化器中使用)。这意味着我们可以验证属性值，但不能验证整个对象，因为我们不知道用户何时完成了属性设置。如果需要这样的验证，就不能使用`init`访问器，值需要作为构造函数参数传递。或者，接受这些类型的 API 必须验证它们。

## 记录

C#类是引用类型。这意味着变量不是该类型的实例，而是对该类型的实例(位于托管堆上)的引用。引用类型相等的默认实现是检查两个变量是否引用同一个实例:

```
class Person { public string FirstName { get; set; }};

Assert.False(new Person { FirstName = "John" }
         .Equals(new Person { FirstName = "John" }));

```

当处理表示数据的对象时，能够比较两个不同的实例并查看它们是否具有相同的值会很有用。这正是新的 C# 9 记录所支持的:

```
record Person { public string FirstName { get; set; }};

Assert.True(new Person { FirstName = "John" }
         .Equals(new Person { FirstName = "John" }));

```

我们已经将`Person`声明从`class`更改为`record`。现在，具有相同值的两个不同实例被视为相等。

`Person` `record`是一个`class`，编译器为其生成了一个基于该类型字段的`IEquatable`和`GetHashCode`实现。在`class`中自己做这件事是可能的，但是很繁琐而且容易出错。

编译器还将生成一个`ToString`方法，该方法返回一个带有类型名、公共字段值和可读属性值的字符串。

您可以向`record`添加构造函数和其他成员，就像您对常规`class`所做的那样。记录使用通常的 `:` 语法支持继承:

```
record Student : Person
{
 public int ID { get; set; }
}

```

相等实现考虑了运行时类型:一个`Person`实例永远不会等于一个`Student`实例。

C#记录可以与`init`访问器一起使用，这提供了一种方便的方式来构建在创建后不能修改的类型(不可变类型):

```
public record Person
{
 public string? FirstName { get; init; }
 public string? LastName { get; init; }
}

```

当处理不可变数据时，通常通过修改现有值来创建新值。记录通过`with`表达式支持这一点:

```
Person jane = person with
{
 FirstName = "Jane"
};

```

`with`表达式从实例中复制值，并允许通过`init`访问器对其进行更改。复制依赖于为`record`类型生成的复制构造函数。

一行程序允许您用构造函数和解构方法创建不可变的记录:

```
record Person(string FirstName, string LastName);

var person = new Person("John", "Doe");
(string firstName, string lastName) = person;

```

可以向记录中添加正文，以添加额外的成员或细化否则会自动生成的属性:

```
record Person(string FirstName, string lastName)
{
 public string LastName { get; } = lastName;
}

```

## 结论

在本文中，我们研究了`init`访问器，我们可以用它来声明不可变的属性。然后，我们看到了`records`如何使构建带有值语义的引用类型变得容易。通过组合`init`访问器和`records`，我们可以构建不可变的数据模型，通过`with`表达式支持变更。

C# 9 系列的第 5 部分涵盖了与[本机互操作和性能](/blog/2021/04/27/some-more-c-9/)相关的高级特性。

C# 9 可以和[一起使用。NET 5 SDK](https://developers.redhat.com/blog/2020/12/22/net-5-0-now-available-for-red-hat-enterprise-linux-and-red-hat-openshift/) ，在[红帽企业版 Linux](/products/rhel/overview) 、[红帽 OpenShift](/products/openshift/overview) 、 [Fedora](http://fedoraloves.net/) 、 [Windows、macOS 等 Linux 发行版](https://dotnet.microsoft.com/download)上都有。

*Last updated: June 17, 2021*
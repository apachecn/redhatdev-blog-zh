# C# 9 模式匹配

> 原文：<https://developers.redhat.com/blog/2021/04/06/c-9-pattern-matching>

我们 C# 9 系列的前一篇文章研究了[顶级程序和目标类型表达式](/blog/2021/03/30/c-9-top-level-programs-and-target-typed-expressions/)。在本文中，我们将介绍模式匹配的新特性。你可以在 [C# 8 模式匹配](/blog/2020/02/27/c-8-pattern-matching/)中找到 [C#](/topics/c) 以前版本提供的语法概述。

## 类型模式

当检查一个类型时，以前版本的 C#要求你包含一个变量名(或者一个`_`丢弃)。这在 C# 9 中不再需要:

```
// is pattern with Type
if (input is Person)
...

// case pattern with Type
switch (input)
{
  case Person:
    ...  

// is pattern with tuple Type
if (input is (int, string))
...

```

## 组合模式

在早期版本的 C#中使用`is`表达式，您已经可以使用常规逻辑操作符组合模式:

```
if (person is Student || person is Teacher)
...

```

然而，这对`switch`表达式和`switch` `case`标签不起作用。C# 9 增加了对使用`and`和`or`关键字组合模式的支持，这对`if`和`switch`都有效:

```
if (person is Student or Teacher)
...

decimal discount = person switch
{
   Student or Teacher => 0.1m,
   _ => 0
};

switch (person)
{
   case Student or Teacher:
      ...

```

`and`模式的优先级高于`or`模式。您可以添加括号来更改或澄清优先级。

## 反转模式

在 C# 9 中，你可以使用`not`关键字来反转模式:

```
if (person is not Student)
...

switch (person)
{
  case not Student:
    ...

```

一个有趣的例子是`is not null`模式。这将检查引用是否不为空。当类型重载了`!=`操作符时，使用`!= null`可能会检查出一些不同的东西。

```
if (person is not null)
...

```

## 关系模式

关系模式允许您将表达式与常量数值进行比较:

```
decimal discount = age switch
{
   <= 2 => 1,
   < 6  => 0.5m,
   < 10 => 0.2m,
   _    => 0
};

```

## 模式中的模式

模式也可以包含其他模式。这种嵌套让您能够以简洁易读的方式表达复杂的条件。以下示例结合了几种类型的模式:

```
if (person is Student { Age : >20 and <30 })
...

```

## 结论

在本文中，我们研究了 C# 9 中新的模式匹配特性。这些附加功能允许您用清晰、简洁的语法表达更复杂的条件。

在下一篇文章中，我们将探索 C# 9 中方法和函数的新特性。

C# 9 可以和[一起使用。NET 5 SDK](/blog/2020/12/22/net-5-0-now-available-for-red-hat-enterprise-linux-and-red-hat-openshift/) ，可以在[红帽企业 Linux](/products/rhel/overview) 和[红帽 OpenShift](/products/openshift/overview) 上，在 [Fedora](http://fedoraloves.net/) 和[上从微软获得，用于 Windows、macOS 和其他 Linux 发行版](https://dotnet.microsoft.com/download)。

*Last updated: April 21, 2021*
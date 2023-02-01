# 使用 Java 10 var 关键字简化局部变量类型定义

> 原文：<https://developers.redhat.com/blog/2018/05/25/simplify-local-variable-type-definition-using-the-java-10-var-keyword>

可能很多人都听说过， [Java 10](http://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html) 于 2018 年 3 月发布。它是 Oracle 公司发布一个短期版本，带有许多新特性和增强功能。Java 10 中的一个重要特性是 l *局部变量类型推断*，在 [JEP (Java 增强提案)286](http://openjdk.java.net/jeps/286) 中有详细介绍。即将于 2018 年 9 月发布的 Java 版本将是 Java 的长期支持(LTS)版本。(注意，一般来说，LTS 每三年发布一次。)

现在让我们看一个 Java 10 *局部变量类型推断*特性的例子。

这个特性的主要优点是减少了样板变量类型定义，增加了代码的可读性。这里有一个例子:

```
String s=new String("Java 10");
Integer int=new Integer(10);

```

一个 Java 开发人员阅读以上两条语句不会有任何问题。然而，作为另一个例子，这里有一些更复杂的语句，写起来有点麻烦:

```
MAP<String,String> map=new HashMap<String,String>(); 
MAP<User,List<String>> listofMovies=new HashMap<>();

```

在 Java 10 中，`var`关键字允许局部变量类型推断，这意味着编译器将推断出局部变量的类型，所以您不需要声明它。因此，您可以替换上面的两个语句，如下所示:

```
var map=new HashMap<String,String>();
var listofMovies=new HashMap<User,List<String>>();
```

以下是 Java 10 中关于局部变量类型推理需要记住的几点:

1.每个包含`var`关键字的语句都有一个静态类型，它是值的声明类型。这意味着分配不同类型的值总是会失败。因此，Java 仍然是一种静态类型语言(不像 JavaScript)，应该有足够的信息来推断局部变量的类型。如果没有，编译就会失败，例如:

```
var id=0;// At this moment, compiler interprets 
//variable id as integer.
id="34"; // This will result in compilation error 
//because of incompatible types: java.lang.String 
//can't be converted to int.

```

注意，JavaScript 也有一个`var`关键字的概念，但这与 Java 10 `var`完全不同。JavaScript 没有变量的类型定义。因此，JavaScript 运行时会成功地解释上面的例子，这也是引入 TypeScript 的原因之一。

2.让我们看一个继承场景。假设从父类`Person`扩展了两个子类(`Doctor`、`Engineer`)。假设有人创建了一个`Doctor`的对象，如下所示:

```
var p=new Doctor(); // In this case, what should be
//the type of p; it is Doctor or Person?
```

注意，在这种情况下，用`var`声明的变量总是初始化器的类型(在这种情况下是`Doctor`)，当没有初始化器时`var`可能不被使用。因此，如果您重新分配上面的变量`p`，如下所示，编译会失败:

```
p=new Engineer(); // Compilation error saying
//incompatible types

```

所以我们可以说*多态行为与* `var` *关键字*不起作用。

3.以下是不能使用局部变量类型推理的地方:

a)不能对方法参数使用局部变量类型推理:

```
public long countNumberofFiles(var fileList);// Compilation 
//error because compiler cannot infer type of local
//variable fileList; cannot use 'var' on variable without 
//initializer

```

b)不能将`var`变量初始化为空。通过赋值 null，不清楚应该是什么类型，因为在 Java 中，任何对象引用都可以是 null。在下面的例子中，因为空值没有预定义的数据类型，编译器无法解释`count`的类型，这将导致复杂错误。

```
var count=null;// Compilation error because 
//compiler cannot infer type for local variable
//count since any Java object reference can be null

```

注意，JavaScript 有一个数据类型 Null，它只能保存一个值:NULL。

c)不能对 lambda 表达式使用局部变量类型推理，因为这些表达式需要显式的目标类型。例如，以下内容会导致编译错误:

```
var z = () -> {} // Compilation error because
//compiler cannot infer type for local variable z;
//lambda expression needs an explicit target type

```

Java 10 `var`旨在为阅读代码的其他开发人员提高代码的可读性。在某些情况下，使用`var`可能会很好；但是，在其他情况下，它会降低代码的可读性。

下面是一个循环遍历`Map`的`entrySet`的例子:

```
Map<String, List<String>> companyToEmployees= new HashMap<>();
  for (Map.Entry<String, List<String>> entry: companyToEmployees . entrySet()) {
      List<String> employees= entry.getValue();
}

```

让我们用 Java 10 `var`重写上面的代码:

```
var companyToEmployees= new HashMap<String, List<String>>();
  for (var entry: companyToEmployees. entrySet()) {
       var employees= entry.getValue();
}

```

从上面的例子可以清楚地看出，使用`var`并不总是好的。

要了解更多关于`var`关键字的信息，我建议浏览一下 [Java 10 局部变量类型参考文档](http://openjdk.java.net/jeps/286)。

*Last updated: June 8, 2021*
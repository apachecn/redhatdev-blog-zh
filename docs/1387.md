# 如何使用 JShell:JDK 9 中引入的命令行工具

> 原文：<https://developers.redhat.com/blog/2017/10/03/use-jshell-command-line-tool-introduced-jdk-9>

很多人可能都知道，甲骨文于 2017 年 9 月 21 日正式发布了 ***JDK 9*** 。在 JDK 5 版和 JDK 8 版之后，从应用程序开发的角度来看，JDK 9 版被认为是最有效的版本。JDK 9 为开发者引入了许多新功能、语言级特性和 API 增强。这篇文章是关于在 JDK 9 中引入的一个名为**T5 的命令行工具 JShellT7。让我们简单地了解一下。**

## JShell 是什么？

1.  ***JShell*** 在 JDK 9 中作为 Java 增强提案(JEP) 222 的一部分被引入。
2.  一般来说，JDK 工具及其命令使开发人员能够处理开发任务，例如编译和运行程序、将源文件打包成 Java 档案(JAR)文件、对 JAR 文件应用安全策略、对应用程序进行故障排除和监控、监视 JVM 静态等。这些工具在 JDK 都是可执行的二进制文件。例子:javac、javap、javah、javadoc、java、jar、jlink、jmod、jdeps、 ***jshell*** (语言 shell)、wsimport 等
3.  JShell 命令行工具将提供一种在 JShell 状态中交互式评估 Java 编程语言的声明、语句和表达式的方法。JShell 状态包括进化的代码和执行状态。为了便于快速调查和编码，语句和表达式不需要出现在一个方法中，变量和方法不需要出现在一个类中。
4.  JShell 是一个 REPL(读取-评估-打印-循环)。对于学习 Java 语言和探索不熟悉的代码(包括新的 Java APIs)来说，这都是非常理想的。读取-求值-打印循环(REPL)是一个交互式编程工具，它循环不断地读取用户输入，计算输入，并打印输入的值或输入引起的状态变化的描述。Scala、Ruby、JavaScript 和 Python 都有 REPLs，都允许小的初始程序。JShell 将 REPL 功能作为 JDK 9 的一部分添加到 Java 平台中。
5.  在 JDK 9 之前，如果开发者想测试一些东西，他必须写一个强制性的测试类。没有办法孤立地测试一个类的功能。Oracle 开发 JShell 的原因之一就是为了处理这种情况。

## 启动 JShell 控制台

一旦你在你的机器上安装了 JDK 9 并且设置了所有的环境变量(比如 JAVA_HOME，PATH 等等)。)，在 cmd 中输入`jshell`就可以启动一个 JShell 控制台。

要启动 JShell:

```
jshell
```

要在详细模式下启动 JShell:

```
jshell -v
```

要退出 JShell 控制台:

```
/exit
```

所以现在我们知道如何启动 JShell 控制台了。让我们看一些例子来帮助我们更好地理解 JShell。

## 例子

### 示例 1

JShell 接受 Java 语句、变量、方法和类定义、导入和表达式。我们将这些 Java 代码片段称为片段。

定义变量:

```
int a=10;
```

打印变量:

```
System.out.println("value of a is ": +a) ;
```

定义和调用方法:

```
void printHello(String name){System.out.println("Hello "+name)};
```

```
printHello("Jshell");
```

```
String concate(String s1, String s2){ return s1+s2;};
```

```
concate("hello","jshell");
```

创建自定义类型:

```
public class Employee{};
```

导入类:

```
import java.io.FileReader;
```

因此，我们可以看到，无论我们在典型的 IDE 中做什么，在 JShell 中同样可以在很短的时间内完成。

### 示例 2

让我们定义一种有助于计算圆面积的方法。

```
public long calculateArea(int radius){ return PI* square(radius)};
```

如果我们在 Jshell 中执行这个代码片段，它不会给出任何编译错误，说`PI`和`square(int)`未定义。`Error`只有当我们试图调用这个方法时才会到来。这个概念在 JShell 中被称为前向引用。

记住这一点，让我们在调用`calculateArea(int r)`之前定义`PI`和`square(int r)`以避免编译错误。

```
double PI=3.14;
```

```
public int square(int n){return n*n;}
```

现在调用方法:

```
calculateArea(5);
```

### 示例 3

JShell 有许多用于控制环境和显示信息的命令。它们通过前导斜杠(/)与代码片段区分开来。您可以通过 ***/vars*** 、 ***/methods*** 、 ***/types* *获得当前变量、方法和类型的信息。*** 你可以用 ***/list*** 命令得到输入片段的列表。要查看导入列表，只需使用 ***/imports。*** 要了解一个命令的更多信息，只需使用 **/help** commandName。

```
/vars
```

```
/methods
```

```
/types
```

```
/imports
```

```
/list
```

```
/help /list
```

```
/help /vars
```

```
/help /exit
```

### 实例 4

#### *搜索机制*

历史搜索是一个强大的工具。在 JShell 中，按下 **Ctrl-r** ，然后按下您希望在历史中搜索的字符串。搜索将向后进行，从您最近的条目开始，包括 jshell 工具以前的会话。

### 实例 5

#### *反馈模式*

反馈模式是 JShell 中一种命名的用户交互配置。有内置模式，用户可以创建自定义模式。内置模式不能修改，但可以复制一份作为用户定义模式的基础。有四种内置模式，按详细程度降序排列:详细、正常、简洁和无声。要设置反馈模式，请使用:

```
/set feedback {modeName}
```

```
/set feedback verbose
```

```
/set feedback silent
```

我将让开发人员尝试这些模式，并尝试理解它们在冗长方面有何不同。

希望这篇文章能帮助你开始使用 JDK 9 中的 JShell。

谢谢！

*Last updated: June 8, 2021*
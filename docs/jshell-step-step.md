# JShell 循序渐进

> 原文：<https://developers.redhat.com/blog/2017/10/24/jshell-step-step>

Java 9 增加了以下新特性:

1.  Java 9 REPL (JShell)
2.  不可变列表、集合、映射和映射的工厂方法。进入
3.  接口中的私有方法
4.  Java 9 模块系统
5.  流程 API 改进
6.  尝试改善资源
7.  可完成的未来 API 改进
8.  反应流

我将在这篇博客中探索 JShell。

要遵循的步骤:

1.  从头开始下载 Java 9 并安装。
2.  运行 shell。
3.  从 Java 9 的 REPL 获得帮助。
4.  在 JShell 中运行一些计算。
5.  定义一个函数并使用它。

步骤 1:下载 Java 9

*   前往:[http://jdk.java.net/9/](http://jdk.java.net/9/)并在 jdk 9 上下载。
*   请验证您是否安装了它。

```
[anijhawa@anijhawa ~]$ java -version
java version "9-ea"
Java(TM) SE Runtime Environment (build 9-ea+165)
Java HotSpot(TM) 64-Bit Server VM (build 9-ea+165, mixed mode)

```

步骤 2:运行 JShell

要连接 jshell，请在终端中键入 jshell。

```
[anijhawa@anijhawa ~]$ jshell
| Welcome to JShell -- Version 9-ea
| For an introduction type: /help intro
jshell>

```

步骤 3:寻求帮助

*   在终端上键入/help，获得一个可以用 JShell 做的事情的列表:

```
jshell> /help
| Type a Java language expression, statement, or declaration.
| Or type one of the following commands:
| /list [<name or id>]
| list the source you have typed
| /edit <name or id>
| edit a source entry referenced by name or id
| /drop <name or id>
| delete a source entry referenced by name or id
| /save [-all] <file>
| Save snippet source to a file.
| /open <file>
| open a file as source input
| /vars [<name or id>]
| list the declared variables and their values
| /methods [<name or id>]
| list the declared methods and their signatures
| /types [<name or id>]
| list the declared types
| /imports
| list the imported items
| /exit
| exit jshell
| /env [-class-path <path>] [-module-path <path>] [-add-modules <modules>] ...
| view or change the evaluation context
| /reset [-class-path <path>] [-module-path <path>] [-add-modules <modules>]...
| reset jshell
| /reload [-restore] [-quiet] [-class-path <path>] [-module-path <path>]...
| reset and replay relevant history -- current or previous (-restore)
| /history
| history of what you have typed
| /help [<command>]
| get information about jshell
| /set editor|start|feedback|mode|prompt|truncation|format ...
| set jshell configuration information
| /? [<command>]
| get information about jshell
| /!
| re-run last snippet
| /<id>
| re-run snippet by id
| /-<n>
| re-run n-th previous snippet
|
| For more information type '/help' followed by the name of a
| command or a subject.
| For example '/help /list' or '/help intro'.
|
| Subjects:
|
| intro
| an introduction to the jshell tool
| shortcuts
| a description of keystrokes for snippet and command completion,
| information access, and automatic code generation
| context
| the evaluation context options for /env /reload and /reset

```

*   从/help 命令中导入一个有趣的命令/imports，请试试这个，看看下面的默认导入:

```
jshell> /imports 
| import java.io.*
| import java.math.*
| import java.net.*
| import java.nio.file.*
| import java.util.*
| import java.util.concurrent.*
| import java.util.function.*
| import java.util.prefs.*
| import java.util.regex.*
| import java.util.stream.*

```

第四步:做一些基本的计算。

```
jshell> 2+2
$1 ==> 4

jshell> 4%7
$2 ==> 4

jshell> 14&3
$3 ==> 2

jshell> $2// we got variable number two!
$2 ==> 4

```

第五步:乘法功能代码

让我们定义一个函数，它接受一个 int 数并对它求平方，然后我们调用它。

```
jshell> int squareThis(int i){return i*i;}
| created method squareThis(int)

jshell> squareThis(5)
$6 ==> 25
```

* * *

要构建您的 Java EE 微服务 **请访问** [**野生蜂群**](https://developers.redhat.com/promotions/wildflyswarm-cheatsheet/) **并下载备忘单。**

*Last updated: November 15, 2018*
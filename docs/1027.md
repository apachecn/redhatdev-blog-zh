# 使用 Byteman 快速诊断 Java 应用程序

> 原文：<https://developers.redhat.com/blog/2018/11/06/diagnosing-java-applications-with-byteman>

生产受到软件问题的影响总是不希望出现的情况。然而，诊断生产问题绝不应该是一项计划外的活动。结构化测试和 QA 工作将理想地防止任何软件错误进入产品。因此，进退两难的问题是如何为生产中未预料到的事情做准备，这些事情在早期的测试和 QA 阶段没有考虑到。

本文讨论了 [Byteman](http://byteman.jboss.org/) ，这是一个工具，它利用 Java Instrumentation API 将 [Java](https://developers.redhat.com/topics/enterprise-java/) 代码注入到方法中，而无需重新编译、重新打包甚至重新部署应用程序。

要解决生产问题，答案在于自适应工具，这些工具允许动态诊断选定的功能，而对系统其余部分的干扰最小。想想最近引入的一个新的软件组件或功能，它作为一个整体，并不总是如预期的那样执行，并且其现有的工具并没有提供关于问题的可能根本原因的所需的洞察力。由于仅仅重新启动一个复杂的应用程序在生产中可能是不可行的，或者使问题在一段时间内消失，所以您需要能够安全地动态修改正在运行的代码，以避免在诊断手头的问题时出现耗时的延迟。

更具体地说，当系统仍在运行并显示问题时，您可以通过为选定的应用程序方法添加计时并监控它们的错误率来开始诊断工作。这将允许您以迭代的方式缩小确切的问题范围，而不会影响应用程序的其他部分。考虑到这些追溯性的代码更改将在生产中完成，修改应该绝对肯定不会引入额外的和可能更严重的问题。这意味着需要使用通用的、经过验证的工具来完成更改，但要以特定于应用程序的方式进行。

[Java Instrumentation API](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html) 允许在运行时修改 Java 虚拟机(JVM)上方法的字节码。虽然从技术上来说，这可以实现对应用程序的任何想要的更改，以了解其行为，但在字节码级别处理不清楚的生产问题将与上述高级迭代方法完全相反。

## 贝特曼来救援了

与许多其他字节码转换器不同，Byteman 是在 Java 而不是字节码层面上运行的。您为 Byteman 提供一个或多个规则，这些规则指定您想要执行的 Java 代码以及您想要将它注入到方法中的位置。Byteman 解决了如何重写字节码的问题，这样它的行为就好像原始 Java 代码包含了您所请求的源代码级别的更改。Byteman 还进行必要的类型检查和类型推断，这对于转换的安全性是绝对必要的。

下面是 Byteman 规则的一个简单示例，通过记录异常计数并以一定的时间间隔打印出来，可以帮助您了解在方法执行期间抛出异常的频率:

```
RULE Count exits via exceptions
CLASS com.example.SomeClass
METHOD someMethod
AT EXCEPTION EXIT
IF true
DO incrementCounter("exceptions");
ENDRULE

RULE Print exception exit count
CLASS com.example.SomeClass
METHOD someMethod
AT ENTRY
BIND exceptions = readCounter("exceptions");
IF exceptions % 10 == 0
DO trace("Exception exit count for someMethod - ");
trace("" + new java.sql.Timestamp(System.currentTimeMillis()));
traceln(": " + readCounter("exceptions"));
ENDRULE
```

`CLASS`、`METHOD`、位置(`AT ENTRY`等的组合。)确定规则中提供的 Java 代码在哪里被注入和执行。出现在`BIND`、`IF`或`DO`子句中的所有表达式都只是普通的旧 Java 代码。必须始终提供一个`IF`表达式来确定`DO`动作何时运行。对内置便利方法`incrementCounter`、`readCounter`、`trace`和`traceln`的调用实际上调用了 Byteman 提供的助手类(恰当地命名为`Helper`)的相应方法。第一对支持在执行期间对事件进行计数，第二对只是包装打印到`System.out`的调用。

总是对`IF`表达式进行类型检查，以确保它是布尔型的。相比之下，规则变量`exceptions`的类型被推断为`int`(使用`readCounter`的已知签名)，并在使用时用于检查类型正确性，例如模(%)或字符串串联(+)操作。

使用 Byteman 助手脚本，对应于这些规则的 Java 语句可以动态地注入到正在运行的 Java 应用程序中，而不会影响应用程序的任何其他部分，除了这一个特定的方法。稍后，这些修改也可以用 Byteman 助手脚本删除。在某种程度上，这已经提供了以前无法从正在运行的应用程序获得的信息，并允许您查看该方法是否按预期执行。

显然，这种最初的方法有一些缺点。针对不同类型的跟踪目的，为几种方法手动编写规则将是乏味且容易出错的。例如，如果利用定制的 Java 类和方法，创建跟踪方法执行时间的规则会更容易。最后但同样重要的是，统计数据不应该写入 stdout，而是应该通过标准的 JMX 接口提供，以便常用的监控工具可以使用它们。

## Byteman 自动化工具

为了使使用 Byteman 快速诊断问题更容易，并解决手动方法的上述缺点，最近引入了一个 [Byteman 自动化工具](https://github.com/myllynen/byteman-automation-tutorial/tree/master/byteman-automation-tool)。

该工具自动生成 Byteman 规则，以提供来自未修改的 Java 应用程序的统计数据，如每个方法的调用次数、方法的执行次数、每个方法的异常退出计数、类的实例数量和实例生命周期。可以动态启用和禁用任何一组统计数据，然后使用标准工具使用 JMX 进行监控。

该工具只需要在一个简单的文本文件中定义目标方法，然后用一组选定的命令行选项生成所需的规则。下面的示例将创建规则，为三种不同的方法提供方法执行时间和异常退出计数的指标:

```
$ cat targets.txt
com.example.SomeClass#methodOne
com.example.SomeClass#methodTwo
com.example.SomeClass#methodThree
$ java \
    -jar ./target/proftool-1.0.jar \
      --input-file targets.txt \
      --register-class com.example.SomeClass \
      --register-method 'methodOne' \
      --call-exectimes-min \
      --call-exectimes-avg \
      --call-exectimes-max \
      --call-exit-except \
      --output-file rules.btm
$ wc -l rules.btm
108
$ tail -n 9 rules.btm
RULE Exits via exceptions from method: com.example.SomeClass - methodThree
CLASS com.example.SomeClass
METHOD methodThree
AT EXCEPTION EXIT
HELPER org.jboss.byteman.automate.proftool.JMXHelper
COMPILE
IF true
DO incrementMethodExitExceptCount($CLASS, $METHOD);
ENDRULE
```

生成的规则采用了工具的专用助手类部分`JMXHelper`。对其实例方法`incrementMethodExitExceptCount()`的调用更新了特定于类/方法的计数器，并通过 JMX 使该值可用。`$CLASS`和`$METHOD`是 Byteman 提供的特殊规则变量，标识规则被注入的方法。也就是说，在这种情况下，当注入的代码运行时，它们的值将是`com.example.SomeClass`和`methodThree`。

现在，我们可以使用 Byteman helper 脚本将所有需要的 Java 代码注入到 JVM 上正在运行的应用程序中，以开始收集这些统计数据:

```
$ bminstall <pid-of-jvm>
$ bmsubmit -s proftool-1.0.jar
$ bmsubmit -l rules.btm
```

(如果 JVM 启动时没有启用 JMX 度量，那么在 [repo](https://github.com/myllynen/byteman-automation-tutorial/blob/master/byteman-automation-tool/JMXEnabler.java) 中有一个简单的实用程序，用于在 JVM 上动态启用 JMX。)

以上是查看这三种方法的最短、平均和最长执行时间，以及查看它们因异常而退出的频率所需的全部内容。像 JConsole 或 Prometheus 这样的工具可以用来分析情况和决定下一步。

## 结论

在本文中，我们看到了 Byteman 自动化工具如何提供一种快速的方法，为未经修改的 Java 应用程序提供额外的工具，甚至不需要重启。然后，该信息可用作进一步故障检修和纠错的基础。

如果需要进一步的细节，Byteman 支持许多替代的“AT”位置，用于注入代码来跟踪和响应应用程序事件，如倒计时、标志和计时器。下面列出的 Byteman 程序员指南和其他资源提供了关于如何使用这些附加功能的完整细节。

## Byteman 资源

*   [字节门主页](http://byteman.jboss.org/)
*   [字节曼文档](http://byteman.jboss.org/docs.html)
*   [字节曼自动化工具](https://github.com/myllynen/byteman-automation-tutorial/tree/master/byteman-automation-tool)
*   [*字节曼程序员指南*](http://downloads.jboss.org/byteman/latest/byteman-programmers-guide.html)
*   [字节曼自动化教程](https://github.com/myllynen/byteman-automation-tutorial)

## 其他 Byteman 文章

*   [使用 Byteman 找出 Java 应用服务器上时区改变的原因](https://developers.redhat.com/blog/2018/02/21/byteman-timezone-changed/)
*   [使用 Red Hat JBoss Fuse 和 AMQ 启用 Byteman 脚本–第 1 部分](https://developers.redhat.com/blog/2018/01/02/enabling-byteman-script-red-hat-jboss-fuse-amq/)
*   [使用 Red Hat JBoss Fuse 和 AMQ 启用 Byteman 脚本–第 2 部分](https://developers.redhat.com/blog/2018/01/18/byteman-jboss-fuse-amq/)

也可以在红帽开发者博客和红帽开发者[企业 Java 页面](https://developers.redhat.com/topics/enterprise-java/)上看到这些 [Java 文章](https://developers.redhat.com/blog/category/java/)。
# 使用 Byteman 找出 Java 应用服务器上时区改变的原因

> 原文：<https://developers.redhat.com/blog/2018/02/21/byteman-timezone-changed>

本文是关于我遇到的一个实际问题，在服务器运行期间，Java 应用服务器(在我的例子中是 JBoss)上的时区发生了意外变化。很难找到任何模式或变化的原因，因为它是由 HTTP 请求触发的。为了调试这个场景，我使用了 Byteman 工具并将脚本注入到 JVM 中。这帮助我确定了问题的根本原因，并提出了共享 JVM 的一些注意事项(比如在 Java 应用服务器上)。

任何应用服务器都被认为是共享的 JVM。JVM 上部署了多个应用程序，它们共享相同的资源。在这种情况下，需要采取一些预防措施。其中之一是处理 JVM 的时区。

Byteman 是一个工具，它使得跟踪、监控和测试 Java 应用程序和 JDK 运行时代码的行为变得容易。它将 Java 代码注入到您的应用程序 API 或 Java 运行时方法中，而不需要您重新编译、重新打包甚至重新部署您的应用程序。注入可以在启动时执行，也可以在运行代码时执行。

Byteman 最简单的用法是注入跟踪应用程序正在做什么的打印语句，通过代码识别控制流并显示静态或实例数据的值。这可以用于监视或调试实时部署，也可以用于检测测试中的代码，以便您可以确保它已经正确运行。通过在非常具体的位置注入代码，您可以避免昂贵的性能开销，这通常会在您打开调试或产品跟踪时出现。

有许多行业，如金融和保险，依赖于部署在应用服务器上的 Java 应用程序来运行他们的日常业务，包括优惠券计算、保险费计算等。所有这些商业计算主要基于日期和时间。如果其中一个应用程序意外更改了任何日期和时间属性，特别是共享 JVM 上的时区(像任何应用服务器一样)，这可能会导致巨大的业务损失或数据异常，这可能很难协调。

通常在 JVM 上，时区总是在启动时设置。那么问题就来了，它如何在运行时改变并影响所有已部署的应用程序？

在一些业务案例中，您可能需要临时操作时区来进行计算。通常的做法是在日历对象中操作它:

```
 Calendar.getInstance().setTimeZone(TimeZone.getTimeZone("UTC"));

e.g.:

 Calendar cal = Calendar.getInstance();
 cal.setTimeZone(TimeZone.getTimeZone("America/Chicago"));
 SimpleDateFormat sdt = new SimpleDateFormat("dd/MM/yyyy HH:mm");
 sdt.setTimeZone(cal.getTimeZone());

```

在这样做的时候，只为这个日历对象设置了时区，没有任何东西会影响 JVM。

然而，有时在不知不觉中(或由于开发人员的错误)，代码可能被写成:

```
 TimeZone.setDefault(TimeZone.getTimeZone("UTC"));

```

这行代码可以覆盖 JVM 上的时区。在执行这一行之后，应用程序将把时区设置为 UTC(根据示例)。因此，所有基于日期/时间的计算都会产生错误的数据。

如果这些更改是在作为 war/jar 文件部署在服务器上的代码库中执行的，并且如果源代码可供审查，我们就可以检查代码。然而，如果这行代码在一个没有源代码的库文件中，就很难找到根本原因。

要进行检查，我们可以遵循以下方法:

1.从[这里](http://byteman.jboss.org/downloads.html)下载最新的 Byteman 二进制文件。

2.在共享 JVM 存在的同一个节点上提取它(即 JBoss 等。).

3.提取之后，创建一个 byteman 脚本，如下所述，并将其保存在应用服务器的类路径中。

```
 RULE check setDefault
 CLASS java.util.TimeZone
 METHOD setDefault(TimeZone)
 AT ENTRY
 IF TRUE
 DO traceStack("XXX attempting to get change the default timezone "+ ": parameter : " +$1 + " : detail : " + $1.getDisplayName())
 ENDRULE

```

4.在您的应用服务器(例如 JBoss)中配置 byteman(如下所示),以便您可以在 standalone.conf 中进行这些更改，其中“extracted path”是您提取 byteman zip 文件的路径，scriptpath 是您在 byteman 脚本中保存 TEMPhas 的路径:

```
JAVA_OPTS="$JAVA_OPTS -javaagent:<extractedpath>/lib/byteman.jar=script:<scriptpath>/examplescript.btm,sys:<extractedpath>/lib/byteman.jar"
```

5.进行上述更改并重新启动服务器。

6.每当调用“TimeZone.setDefault (TimeZone)”方法时，它将打印一个堆栈跟踪来显示调用。这是您跟踪调用并到达导致共享 JVM 上的问题的代码的方法。

此示例/故障排除来自一个真实的使用案例，在该案例中，由于时区的变化，保费计算受到了干扰。在最初的问题中，代码在更改时区(如 UTC)后没有计算保险费。保险费的计算推迟了一天。这给组织造成了一天的损失。

类似地，Byteman 可以帮助调试由部署的应用程序引起的 JVM 上的问题，而无需对部署中的服务器进行任何更改。如上所述，您可以调试问题并找到导致问题的代码/库。作为最佳实践，开发人员应该了解 Java 提供的日期和时间 API，并在代码中明智地使用它们。否则，应用程序中引入的错误可能会给开发该应用程序的企业带来巨大的损失。
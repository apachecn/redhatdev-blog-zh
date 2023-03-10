# OpenJDK 8u 中的 JDK 飞行记录器入门

> 原文：<https://developers.redhat.com/blog/2020/08/25/get-started-with-jdk-flight-recorder-in-openjdk-8u>

OpenJDK 8u 262 版本包括几个安全相关的补丁和一个新的附件，JDK 飞行记录器(JFR)。这篇文章介绍了 OpenJDK 开发人员使用 JDK 飞行记录器与 JDK 任务控制和相关的工具。我还将简要地向您介绍汉堡项目，也被称为集装箱 JFR。

## 关于 JDK 飞行记录器

JDK 飞行记录器是一个故障排除、监控和分析框架，它深深嵌入在 Java 虚拟机(JVM)代码中。它最初是作为 [JEP 328](http://openjdk.java.net/jeps/328) 的一部分在 OpenJDK 11 中引入的。在 OpenJDK 11 之前，JDK 飞行记录器作为商业特性仅在 JRockit 中可用，后来在 Java 开发工具包(JDK)的 Oracle 发行版中也可用。自从 JFR 作为 OpenJDK 11 中适当的开源组件发布以来，越来越多的 [Java](https://developers.redhat.com/topics/enterprise-java) 社区成员希望在旧版本中提供该特性。

2019 年，在 [FOSDEM 的 Java DevRoom](https://archive.fosdem.org/2019/schedule/event/imc/) 和[OpenJDK 提交者研讨会](https://openjdk.java.net/workshop)期间，一群 open JDK 提交者决定组建一个联合任务组，目标是对 OpenJDK 8u 进行必要的更改和修复。一年多一点的时间和许多许多补丁之后，该项目最终与主要的上游 OpenJDK 8u 开发树合并。

第一个公开发布的是 OpenJDK 8u 262 但是，如果你试图自己编译 OpenJDK，你会发现 OpenJDK 8u 262 在编译时默认跳过 JFR。open JDK 8u 272(10 月到期)将是第一个默认编译 JFR 的版本。

**注意**:通过[红帽企业 Linux](https://developers.redhat.com/topics/linux) 或 [Fedora](https://getfedora.org/) 消费 OpenJDK 的开发者会得到一个红帽包管理器(RPM)文件，其中包含对 JFR 的支持。我们当然希望听到您的经历，尤其是关于您发现的错误或问题。

### 引擎盖下的 JDK 飞行记录器

JDK 飞行记录器由两个主要部分组成:一个是包含数据的关键部分，另一个是记录和暴露数据的内部基础设施。这个数据是通过一个叫做 [*事件*](https://developers.redhat.com/topics/event-driven/) 的概念抽象出来的。事件可以有多种有用的相关信息，可以表示时间样本、单触发事件或给定的持续时间。作为开发人员，您可以向事件定义中添加元数据和其他上下文信息，并使用这些信息为您的分析工具描述事件，还可以使其自描述，以便其他人更好地理解事件类型。例如，您可能希望在发生文件访问或垃圾收集(GC)压缩阶段开始时得到通知，或者希望知道完整的垃圾收集阶段需要多长时间。这样的事件可能包含可以被注释的字段——例如，表示一个[周期](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.jfr/jdk/jfr/Period.html)或者一个[频率](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.jfr/jdk/jfr/Frequency.html)——JFR 的工具让你在分析过程中以一种特殊的方式可视化。OpenJDK 8u 包括 160 多个事件供您记录和分析。

尽管这些事件在发生时会被记录下来，但 JFR 本身并不是一个实时工具，也不会在调用点传输事件。(在后来的 OpenJDKs 中有一个 [JFR 事件流 API](https://openjdk.java.net/jeps/349) ，但它的目的不是实时流事件。)相反，底层框架将事件存储在线程本地缓冲区中，然后将这些事件写入全局环形缓冲区。当这些缓冲区被填满时，它们最终会被一个定期线程刷新到磁盘，使用的机制类似于事务数据库使用的机制。

注意:虽然 JFR 的设计可能看起来过于复杂，但它允许有效利用内存和 CPU。一般来说，JFR 的开销非常低——大约 1%，在大多数情况下甚至更低。低开销意味着 JFR 可以在生产时使用，不像大多数其他解决方案那样运行时成本过高。

### JFR 唱片公司

JFR 的记录文件是所有事件及其元数据的二进制表示。这些信息被分成块。*组块*是 JFR 记录中最小的独立信息单元，可以单独读取，但仍能完整描述组块中包含的事件。

所有信息都编码为 LEB128 编码整数，包括引用常量池位置的字符串。这种编码保证了每个记录中的高水平数据压缩。您还可以使用 GZip、LZMA 和 XZ 或 LZ4 等方法进一步压缩录像。根据配置，录像会根据请求或在程序终止时写入磁盘。您还可以有无休止的记录，每隔一段时间写入磁盘，让您看到应用程序的行为随着时间的推移。简而言之，JFR 的配置选项是灵活的。

## 如何使用 JDK 飞行记录器

默认情况下，OpenJDK 中有许多机制可用于控制 JDK 飞行记录器，这使得适应手边的用例变得极其简单。第一种选择是直接用 JVM 启动 JFR，例如:

```
$ java -XX:StartFlightRecording your.application.ClassName
```

您可以使用逗号分隔的选项列表来进一步配置 JFR。例如，您可能希望在退出时转储记录:

```
$ java -XX:StartFlightRecording=dumponexit=true your.application.ClassName
```

您可以使用`jcmd`，一个您可能已经熟悉的实用程序，在应用程序启动后控制 JFR:

```
$ jcmd <pid>

<pid>:
The following commands are available:
VM.unlock_commercial_features
JFR.configure
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
VM.classloader_stats
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.finalizer_info
GC.heap_info
GC.run_finalization
GC.run
VM.uptime
VM.dynlibs
VM.flags
VM.system_properties
VM.command_line
VM.version

help

For more information about a specific command use 'help <command>'.

```

直观地说，`jcmd`实用程序允许您开始和停止记录、配置记录设置、检查记录状态以及转储记录。可以同时运行多个录像。

您可以使用标准的 [Java 飞行记录器 API](https://docs.oracle.com/en/java/javase/14/jfapi/flight-recorder-api-programmers-guide.pdf) 直接从您的应用程序代码中访问记录。在使用 API 时，有些东西是必须要理解的，所以我将进一步讨论这个选项，并在文章的最后展示一个例子。

另一个，可以说是更有用的检索记录的方法是通过 [JDK 任务控制](http://jdk.java.net/jmc/)，一个专门设计来控制和分析记录的应用程序。

**注意**:您可能已经注意到了可用的`jcmd`命令列表中的`unlock_commercial_features`标志。重要的是要意识到 JFR 是*而不是*open JDK 中的一个商业特性。然而，在 JDK 11 之前的任何甲骨文 JDK 中，它都是一个商业特性。出于兼容性的原因，我们保留了这个标志，但是它什么也不做，您可以放心地忽略它。

## JDK 飞行控制中心使用 JDK 飞行记录器

OpenJDK 包含一个名为`jfr`的简单工具，允许您阅读 JFR 记录并从中获得有用的指标。然而，当你把 JFR 录音和 JDK 任务控制(JMC)结合起来时，你会看到它的真正好处。JMC 已经可以在 Fedora 和[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/)(RHEL)7 中通过[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview)(RHS cl)获得，在 RHEL 8 中通过模块获得，Windows 用户可以从 [OpenJDK 开发者门户](https://developers.redhat.com/products/openjdk/download)获得。你也可以通过像 [AdoptOpenJDK](https://adoptopenjdk.net/jmc) 这样的下游发行版获得 JDK 任务控制。

如果你有一个旧的 JMC 安装，你可能会看到一个警告对话框时，试图访问一个 JDK 飞行记录器的 OpenJDK 8u 版本，询问你是否正在使用商业功能。正如我之前提到的，您可以在 OpenJDK 上忽略这条消息(而在 OpenJDK 上*只有*)。这个错误已经在 JDK 任务控制的更高版本中被修复。图 1 显示了商业特征警告。

[![A screenshot of the commercial features warning.](img/275962267254ef066d58894f4ebcab3c.png "Commercial Features Warning")](/sites/default/files/blog/2020/07/openjdk-jfr-1.png)Commercial Features Warning

Figure 1: You can ignore the commercial features warning on any OpenJDK build.

## 演示:分析 GC 分配

JDK 任务控制项目负责人 Marcus Hirt 已经准备了一套很好的教程和演示来探索 JFR 和 JMC 的结合。在本节中，我将引用他的代码，而不是创建一个新的演示。特别是，我将使用他的 GC 分配行为的例子`04_JFR_GC`，来展示 JMC 自动分析数据并提出改进建议的能力。JMC 的分析基于一个叫做*规则引擎*的特性。规则引擎目前正在为 JMC 8.0 进行[大修](https://wiki.openjdk.java.net/display/jmc/Rules+2.0)，以便添加更多的分析选项，提供更好的 API 供工具直接使用，并提高整体性能。

GC 演示程序简单地分配大量数据并将其存储在一个映射中。然后，它在每个分配周期检查映射的内容。尽管真实世界的程序会对数据做一些更有趣的事情，但该模式代表了一个非常典型的哈希映射用例。在我们的例子中，这个程序似乎运行得很好，我们没有遇到任何内存不足的错误或其他类型的错误。这使得演示成为探索隐藏的性能问题和检查可能的瓶颈和优化的完美候选。

### 使用模板

JMC 和 JFR 有一个叫做*模板*的便利功能，允许你用默认设置和事件开始录音。例如，当通过`jcmd`检索记录时，这些模板对应于您可以通过命令行界面(CLI)传递的配置。然而，图形用户界面(GUI)使理解设置变得容易得多。我们将为这个实验选择**剖析**模板和默认的一分钟记录会话。这为演示提供了足够的数据。

如图 2 所示，运行应用程序并检索记录，可以直接回答我们想要立即优化的内容，而无需进一步研究。应用程序进行大量的原语到对象的转换，JMC 告诉你这些分配发生在哪里。

[![Automated Analysis View](img/455f7afad6c61cc6a645b09ae058fbca.png "Automated Analysis View")](/sites/default/files/blog/2020/07/openjdk-jfr-2.png)Automated Analysis View

Figure 2: JDK Mission Control's Automated Analysis view detects many issues automatically.

这个演示是几年前在 Java One 上为一次实践会议创建的，当时的 JMC 版本没有高级分析选项。会议鼓励学生探索**内存**和 **TLAB** 选项卡，以获得内存压力的更详细指示，如图 3 所示。

[![TLAB Allocation View](img/ed6d2c5f06e655577aef42bfebc66331.png "TLAB Allocation View")](/sites/default/files/blog/2020/07/openjdk-jfr-3.png)TLAB Allocation View

Figure 3: Memory allocations seen in the TLAB tab.

**注意**:JMC 7 和更高版本中的分析页面提供了更多的信息，并且总是随着更多的规则和优化策略而改进。JMC 还提供了分析页面作为一个独立的组件导出为 HTML 页面。您可以轻松地将 JMC 分析集成到应用程序中，而无需使用完整的 IDE。当与 JFR API 一起使用时，JMC 的独立分析组件可以让您将强大的监控和分析解决方案集成到您的基础架构中，同时保持极低的内存开销。

### 用 JFR 的旧对象样本事件进行内存剖析

传统上，为了进行有效的内存分析，您需要随着时间的推移访问和探索完整的堆转储，以检查 GC 根和分配历史。另一个同样昂贵的选择是使用像代理这样的方法，通过 Java 本地接口(JNI)对对象分配进行采样。然而，这并不总是可能的，尤其是考虑到完整堆转储中包含的敏感信息。JFR 也可以在这方面提供帮助，这要感谢它的旧对象示例事件，该事件被反向移植到 OpenJDK 8u。

**注**:如果你有兴趣了解更多关于 JFR 老物件样本事件的信息，我再次向你推荐一篇由 Marcus Hirt 撰写的[博客文章。你真应该看看他的博客。这是一个关于侧写、JMC 和 JFR 的信息、技巧和细节的不可思议的来源。](http://hirt.se/blog/?p=1055)

我们将使用另一个独立的小例子来探索 JFR 的旧对象样本事件。当你阅读代码的时候，这是一个相当明显的例子，但是对于探索我们的选择仍然是非常好的。

```
public class Leaks {

   private static final Map<Object, Object> *SESSION_DATA* = new HashMap<>();

   public static class UserInformation {
      private byte[] data = new byte[10000];
   }

   public static void main(String[] args) {
       String userId = "user";
       while (true) {
           UserInformation user = (UserInformation) *SESSION_DATA*.get(userId);
           if (user == null) {
               user = *findUserInformation*(userId);
               *// SESSION_DATA.put(userId, user); // Correct*
*SESSION_DATA*.put(user, user);      *// Wrong*
}
           *sleep*();
       }
   }

   private static UserInformation findUserInformation(String userId) {
       *sleep*();
       return new UserInformation();
   }

   private static void sleep() {
       try {
           Thread.*sleep*(1);
       } catch (InterruptedException e) {}
   }
}

```

代码中突出显示的错误是快速测试在投入生产之前发现的那种错误，但是为了举例，让我们假设这是代码中的一个错误，它进入了生产。图 4 显示了一个 JFR 会话，其中我们打开了旧对象样本事件分析。(在此会话期间，没有堆转储受到损害。)

[![A screenshot of an Old Object Sample Event shown in the Automated Analysis view](img/6d854b320f18183756e9d4a4c2f7fcd5.png "Memory Profiling")](/sites/default/files/blog/2020/07/openjdk-jfr-4.png)Memory Profiling

Figure 4: An Old Object Sample Event is shown in the Automated Analysis view.

分析立即告诉我们去哪里找:一个哈希映射被一遍又一遍地填充，不仅包含越来越多的对象，而且内存分配也很高。即使没有阅读代码，您也会认为这个地图是在一个没有太多控制的循环中填充对象。如图 5 所示，**内存**选项卡显示了更多信息。

[![](img/add590fe4251be25fedaad91028ae5c2.png "openjdk-jfr-5")](/sites/default/files/blog/2020/07/openjdk-jfr-5.png)

Figure 5: The Live Object page is shown side-by-side with the application code. Note the matching line numbers.

在这个选项卡中，我们看到程序代码在**活动对象**页面旁边。带有行号的堆栈跟踪指出了问题所在。

### 关于剖析的一个注记

能够跟踪对象分配和保留是分析内存问题时最重要的工具之一。我记得有一个很难修复的 bug，因为创建的对象数量非常大，但是只有在通过远程 X11 连接运行应用程序的 UI 时才会出现这种情况。此外，每次用户移动鼠标或单击按钮时都会创建对象，导致许多方法重新计算图形界面的位置，但只是在某些情况下。

这两个行为被联系起来是因为在我们处理远程连接的方式上有一个缺陷:计算它们的位置需要知道对象在屏幕上的相对位置。因为这是一个远程连接，所以大量 X11 原子被创建并通过线路来回传递。如果用户运行多个应用程序，这将意味着更多的流量。Java 代码最终会截取这些原子，创建一个 Java 表示，进行更多的计算，然后重复。

当时，我们还没有 JMC，但是有一个类似的工具叫做[恒温器](http://icedtea.classpath.org/thermostat/)。我们使用我们与 [Byteman](https://byteman.jboss.org/) 的集成来创建一个脚本，以分析这些对象是在哪里创建的，以及为什么导致创建这些对象的代码路径被不同地执行。常规的方法分析器很难做到这一点，因为它们倾向于聚合结果。直接从 JFR 录音中获得这些信息非常重要，可以节省我们的时间。更重要的是，考虑到客户可以简单地将他们部署的记录发送给您，而不是让您尝试在本地重现错误、安装更多工具、打开端口、启动代理等等。在这种情况下，记录是所有需要的。

## JDK 飞行记录器 API

前面，我提到过 JFR 有一个内部 API。API 位于`jdk.jfr`名称空间下，包含允许您管理记录和为您的应用程序创建定制事件的类。

您可以编写的最简单的程序是检查 JFR 是否可用:

```
public class CheckJFR {
   public static void main(String[] args) {
       boolean isAvailable = FlightRecorder.*isAvailable*();
       System.*err*.println(isAvailable);
   }
}

```

然后，您可以使用 API 从您的应用程序中以编程方式启动和停止记录。例如，下面的类是创建 JFR 管理器的抽象:

```
import java.io.File;
import java.nio.file.Path;
import java.util.HashMap;
import java.util.Map;
import jdk.jfr.Configuration;
import jdk.jfr.Recording;

public class LocalJFR {
   private Map<Long, Recording> recordings = new HashMap<>();

   @Override
   public long startRecording(String configName) throws Exception {
       Configuration c = Configuration.*getConfiguration*(configName);
       return startRecording(new Recording(c), "jfr-recording");
   }

   @Override
   public long startRecording(String configName, String recordingName)
       throws Exception
   {
       Configuration c = Configuration.*getConfiguration*(configName);
       return startRecording(new Recording(c), recordingName);
   }

   @Override
   public long startRecording() throws Exception {
       return startRecording(new Recording(), "jfr-reopenjdk-jfr-2cording");
   }

   public long startRecording(Recording recording, String name)
       throws Exception
   {
       long id = recording.getId();  
       Path destination = File.*createTempFile*(name + "-" + id,
                                              ".jfr").toPath();
       recording.setDestination(destination);
       recordings.put(id, recording);
       recording.start();
       return id;
   }

   public File endRecording(long id) throws Exception {
       Recording recording = recordings.remove(id);
       recording.stop();
       recording.close();
       return recording.getDestination().toFile();
   }
}

```

虽然这是您可以定义的最简单的事件:

```
@Label("Basic Event")
@Description("An event with just a message as payload")
public class BasicEvent extends Event {
   @Label("Message")
   public String message;
}
```

### 以编程方式创建和监控 JFR 事件

《OpenJDK 中的 JFR》的作者之一 Eric Gahlin 使用 [JFR API](https://github.com/flight-recorder/samples) 整理了一份完整的演示和小型测试列表。该 API 是从 OpenJDK 11 开始的 Java 规范的一部分，但不是 OpenJDK 8 规范的一部分，因此不是所有 OpenJDK 实现都可以访问它。

为了便于版本之间的移植和迁移，我们用一个空的实现创建了一个简单的 [compat-jfr](https://github.com/rh-jmc-team/openjdk8-jfr-compat) 。这个包允许用户检测他们的代码，创建自定义事件，并使用 API 来管理记录。然而，实现是空的，所以方法不做任何事情，事件不提交到内存或磁盘，当查询时，JFR 报告为不可用，不能启动。该应用程序将正常运行、编译和运行，并且具有很好的兼容性。您可以使用 compact-jfr 作为命令行的依赖项，或者将其添加到您的 JDK 的`jre/lib/ext`目录中。

除了通过事件 API 创建自定义事件之外，您还可以在事后检测您的代码，以将事件添加到正在运行的应用程序中。JMC 对此也有一个方便的工具，有一个聪明的名字叫[代理](https://wiki.openjdk.java.net/display/jmc/The+JMC+Agent)。JMC 代理使用一组配置来定义事件，然后用它们来检测正在运行的代码。会话结束后，将移除检测。如果你熟悉 Byteman(你应该很熟悉)，Agent 非常相似，但它不是一个完全的图灵完整语言，而是专注于 JFR 事件。范围的缩小使我们可以专注于用更好的工具来检测 JFR 的问题，这也部分解决了安全性和权限等问题。我们还在开发一个 JMC 插件来控制和配置代理，这是一个正在进行的工作，但已经很有用了[你可以在这里找到它](https://github.com/rh-jmc-team/jmc-agent-plugin)。

## 将 JFR 用于集装箱(汉堡项目)

本文中描述的所有工具都非常棒，因为它们让您可以针对您的特定部署微调 JDK 飞行记录器。然而，我们意识到仍然有大量的工作需要开发者在[容器](https://developers.redhat.com/topics/containers/)中使用 JFR。首先，JFR 的接收端需要通过 Java 管理扩展(JMX)建立一个开放的连接。当然，这种连接可以(也应该)是安全的，但是像[Red Hat open shift Container Platform](https://developers.redhat.com/products/openshift/getting-started)(OCP)这样的容器平台可能不允许或者很难让内部端口对外部世界开放。如果不使用更高级的工具，同时跟踪多个流程也很复杂。OpenShift 有部署控制台来帮助您完成这项任务，但是仍然需要一个更通用的解决方案。

为此，我们创建了一个名为[集装箱 JFR](https://github.com/rh-jmc-team/container-jfr) 的项目，也称为汉堡项目。容器 JFR 是一个简单的三层应用程序，包含一个控制器代理，它通过容器内的 JMX 连接到各种应用程序，并向外部世界公开一个 web 服务接口。JMX 连接可以隐藏在容器中——甚至在非容器环境中，即可以在防火墙后面——而 web 服务接口通过身份验证来保护。该接口允许您从同一个端点控制多个 JVM，因此非常适合多部署。

另一个组件是使用 web 服务的 web UI。它简化了管理，但最重要的是集成了 JMC 的自动分析功能，因此您可以立即看到[应用程序性能](https://developers.redhat.com/blog/category/performance/)，并且只有在分析指出某些问题时才决定下载记录。该项目还包含一个 [Grafana](https://developers.redhat.com/blog/2020/07/10/generate-automated-grafana-metrics-dashboards-for-microprofile-apps/) 数据源，让我们可以在浏览器中创建图表(例如，用户可以在他们的仪表板中集成记录)；一个实验性的[普罗米修斯](https://prometheus.io)出口商(这不是消费录音的最佳方式，但仍然有用)；最后但同样重要的是，一套全面的 OpenShift 或 Kubernetes 操作者 API。使用这些操作 API，您只需简单地点击一下鼠标，就可以安装、运行和配置项目。

**注意** : Gunnar Morling 写了一篇关于[使用定制的、特定于应用程序的 JFR 事件来监控 REST API](https://github.com/gunnarmorling/jfr-custom-events) 的博文。这篇文章展示了流式 API 和自定义 JFR 事件，所以我会带你去那里了解更多细节。贡纳是最棒的！

## 结论

JDK 飞行记录器是第一个可用于 OpenJDK 的监控和分析工具，可以在不增加运行时系统负担的情况下公开如此高级别的信息。JFR 提供这种级别的信息，因为它与 JVM 深度集成。能够使用事件 API 或代理工具创建自定义事件也让您能够从应用程序的角度利用 JFR，而不仅仅是从运行时的角度。

OpenJDK 是对公众的巨大贡献，JDK 飞行记录器可以说是 OpenJDK 开源以来最重要的贡献。当 Oracle 开源 JDK 飞行记录器和 JDK 任务控制时，他们为 Java 社区做出了不可思议的贡献，这应该得到承认。OpenJDK 8u 的后向移植最终将这个基础设施带到了 OpenJDK 所有积极维护的版本中。

尽管我们希望您已经迁移到更高版本的 OpenJDK，以从所有附加功能和性能改进中受益，但工具箱中增加的 JFR 将帮助您的应用程序在任何版本的 OpenJDK 上执行得更好、更快、更顺利。

## 承认

我要感谢:

*   Marcus Hirt 作为 JDK 任务控制项目的项目领导所做的工作。在社区参与方面，他确实树立了很高的标准，他的博客是灵感和知识的不可思议的来源。
*   Gunnar Morling 在 JFR 容器开发的早期帮助和测试，以及他的反馈和建议。
*   红帽的 JDK 任务控制团队为 JMC 做出的惊人贡献，以及他们在《特工》、《JFR 契约》和《JFR 集装箱》上的工作。

最后，非常感谢最初的 JDK 飞行记录器团队提供了这项神奇的技术，并感谢 Oracle 开源了这项技术。说到神奇，你知道 JMC [赢得了 2020 年最佳 Java 特性](http://hirt.se/blog/?p=1230)大赛吗？

## 额外资源

这里是本文中提到的资源，以及到演示文稿、文章和源代码的有趣的附加链接，您可以使用它们来了解更多关于 JDK 飞行记录器和 JDK 任务控制的信息:

*   【Java 任务控制和飞行记录器中间件应用监控简介 (FOSDEM 演示，2019)
*   [JMC & JFR—2020 年远景](https://fosdem.org/2020/schedule/event/imc/) (FOSDEM 演示文稿，2020 年)
*   关于 OpenJDK 8 的 [JFR 兼容性 API 的更多信息](https://github.com/rh-jmc-team/openjdk8-jfr-compat)
*   使用 Java Mission Control 进行低开销方法分析 (Marcus Hirt，2013)
*   [压缩飞行记录](http://hirt.se/blog/?p=1166)(马库斯·赫特，2019)
*   [使用带有 OpenJDK 11](https://dzone.com/articles/using-java-flight-recorder-with-openjdk-11-1) 的 Java 飞行记录器(Laszlo Csontos，2018)
*   更多关于[集装箱 JFR 项目](https://github.com/rh-jmc-team/container-jfr)
*   更多关于 [JDK 任务控制](https://github.com/openjdk/jmc)
*   源代码和示例，用于理解如何使用 JDK 飞行记录器创建和使用[自定义事件(Gunnar Morling，2020)](https://github.com/gunnarmorling/jfr-custom-events)
*   飞行记录器示例:演示如何使用 JDK 飞行记录器 API 的代码片段
*   jmc-jshell :试验 JDK 飞行记录器和 jmc 核心类的更简单的方法
*   JmFrX 简介:一个用 JDK 飞行记录器捕捉 JMX 数据的小工具 (Gunnar Morling，2020)

*Last updated: April 7, 2022*
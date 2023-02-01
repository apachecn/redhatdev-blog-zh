# Quarkus:使“helloworld”现代化 JBoss EAP 快速入门，第 2 部分

> 原文：<https://developers.redhat.com/blog/2019/11/08/quarkus-modernize-helloworld-jboss-eap-quickstart-part-2>

在本系列的第一部分[中，我们详细了解了](https://developers.redhat.com/blog/?p=642737)[Red Hat JBoss Enterprise Application Platform(JBoss EAP)quick starts`helloworld`](https://github.com/jboss-developer/jboss-eap-quickstarts/tree/7.2.0.GA/helloworld)quick start，作为理解如何使用 Quarkus 支持的技术(CDI 和 Servlet 3)更新 Java 应用程序的起点。在这一部分，我们将继续讨论现代化，看看内存消耗。

在处理现代化过程时，测量性能是一个基本主题，内存消耗报告是性能分析的一部分。从一开始就使用这些工具是值得的，这样它们就可以用于评估现代化过程中实现的改进。

关于测量内存使用的详细介绍，请参见[测量性能——我们如何测量内存使用？](https://quarkus.io/guides/performance-measure#how-do-we-measure-memory-usage)夸尔库斯指南。

在下面的段落中，将使用 [`pmap`](https://linux.die.net/man/1/pmap) 和 [`ps`](https://linux.die.net/man/1/ps) 工具在 *Linux* 系统中捕获上述三种不同应用程序风格(JBoss EAP、打包 JAR 和本机可执行文件)的内存消耗数据。

### JBoss EAP

按照“[部署`helloworld`](#deploy-helloworld) ”一节启动 JBoss EAP 实例，并使用以下命令检索其 PID(例如`7268`):

```
$ pgrep -lf jboss
7268 java

```

**注意:**添加`-a`选项来检索完整的命令行(即`$ pgrep -af jboss`)。

现在，`7268` PID 可用于执行以下两个命令:

```
$ ps -o pid,rss,command -p 7268
PID RSS COMMAND 
7268 665348 java -D[Standalone] -server -verbose:gc -Xloggc:/home/mrizzi/Tools/jboss-eap-7.2.0/jboss-eap-7.2/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms1303m -Xmx1303m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferI

```

并且:

```
$ pmap -x 7268
7268:   java -D[Standalone] -server -verbose:gc -Xloggc:/home/mrizzi/Tools/jboss-eap-7.2.0/jboss-eap-7.2/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms1303m -Xmx1303m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -Dorg.jboss.boot.log.file=/home/mrizzi/Tools/jboss-eap-7.2.0/jboss-eap-7.2/standa
Address           Kbytes     RSS   Dirty Mode  Mapping
00000000ae800000 1348608  435704  435704 rw---   [ anon ]
0000000100d00000 1035264       0       0 -----   [ anon ]
000055e4d2c2f000       4       4       0 r---- java
000055e4d2c30000       4       4       0 r-x-- java
000055e4d2c31000       4       0       0 r---- java
000055e4d2c32000       4       4       4 r---- java
000055e4d2c33000       4       4       4 rw--- java
[...]
ffffffffff600000       4       0       0 r-x--   [ anon ]
---------------- ------- ------- -------
total kB         3263224  672772  643024

```

评估`RSS`值，看起来 JBoss EAP 的内存消耗大约是 650MB。

### 包装罐

参考“[运行`helloworld`打包 JAR](#run-the-helloworld-packaged-jar) ”一节，通过执行以下命令启动打包 JAR 应用程序:

```
$ java -jar ./target/helloworld-<version>-runner.jar

```

并再次使用`pgrep`命令检索其 PID(这一次使用上面注释中描述的`-a`选项):

```
$ pgrep -af helloworld
6408 java -jar ./target/helloworld-<version>-runner.jar

```

按照相同的过程，使用`6408` PID 通过执行以下命令来评估内存消耗:

```
$ ps -o pid,rss,command -p 6408
  PID   RSS COMMAND
 6408 125732 java -jar ./target/helloworld-quarkus-runner.jar

```

并且:

```
$ pmap -x 6408
6408:   java -jar ./target/helloworld-quarkus-runner.jar
Address           Kbytes     RSS   Dirty Mode  Mapping
00000005d3200000  337408       0       0 rw---   [ anon ]
00000005e7b80000 5046272       0       0 -----   [ anon ]
000000071bb80000  168448   57576   57576 rw---   [ anon ]
0000000726000000 2523136       0       0 -----   [ anon ]
00000007c0000000    2176    2088    2088 rw---   [ anon ]
00000007c0220000 1046400       0       0 -----   [ anon ]
00005645b85d6000       4       4       0 r---- java
00005645b85d7000       4       4       0 r-x-- java
00005645b85d8000       4       0       0 r---- java
00005645b85d9000       4       4       4 r---- java
00005645b85da000       4       4       4 rw--- java
[...]
ffffffffff600000       4       0       0 r-x--   [ anon ]
---------------- ------- ------- -------
total kB         12421844  133784  115692

```

评估`RSS`值，看起来打包的 JAR 内存消耗大约是 130MB。

### 本机可执行文件

在这种情况下，在“[运行`helloworld`本机可执行文件](#run-the-helloworld-native-executable)”部分之后，可以使用以下命令启动本机可执行应用程序:

```
$ ./target/helloworld-<version>-runner

```

它的 PID 可以用前一种情况下使用的相同命令来检索:

```
$ pgrep -af helloworld
6948 ./target/helloworld-<version>-runner

```

然后，使用带有`ps`和`pmap`命令的`6948` PID:

```
$ ps -o pid,rss,command -p 6948
  PID   RSS COMMAND
 6948 19084 ./target/helloworld-quarkus-runner

```

并且:

```
$ pmap -x 6948
6948:   ./target/helloworld-quarkus-runner
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000      12      12       0 r---- helloworld-quarkus-runner
0000000000403000   10736    8368       0 r-x-- helloworld-quarkus-runner
0000000000e7f000    7812    6144       0 r---- helloworld-quarkus-runner
0000000001620000    2024    1448     308 rw--- helloworld-quarkus-runner
000000000181a000       4       4       4 r---- helloworld-quarkus-runner
000000000181b000      16      16      12 rw--- helloworld-quarkus-runner
0000000001e10000    1740     156     156 rw---   [ anon ]
[...]
ffffffffff600000       4       0       0 r-x--   [ anon ]
---------------- ------- ------- -------
total kB         1456800   20592    2684

```

评估`RSS`值，看起来本机可执行内存消耗大约为 20MB。

### 内存消耗比较

总之，检索到的内存消耗数据如下:

*   JBoss EAP: 650MB
*   打包的 JAR: 130MB
*   本机构建器:20MB

因此，很明显，由于运行本机可执行文件，在内存使用方面有优势。

## 结论

在这些文章中，我们探讨了如何使用 Quarkus 支持的技术(CDI 和 Servlet 3)来更新 Java 应用程序，并概述了开发、构建和运行应用程序的不同方法。我们还展示了如何捕获内存消耗数据，以便评估在现代化过程中实现的改进。这些概念为理解 Quarkus 如何工作以及它为什么如此有用提供了基础，无论该应用是简单的快速入门还是更复杂的生产级应用。

*Last updated: July 1, 2020*
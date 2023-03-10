# Debuginfod 项目更新:新客户和指标

> 原文：<https://developers.redhat.com/blog/2021/02/25/debuginfod-project-update-new-clients-and-metrics>

距离我们上一次更新`debuginfod`已经过去一年了，这是一个 HTTP 文件服务器，为类似调试器的工具提供调试资源。从那时起，我们一直忙于在一系列开发工具上集成客户端，并改进服务器的可用指标。这篇文章涵盖了自上次更新以来我们添加到`debuginfod`中的特性和改进。

**注**:关于`debuginfod`和如何使用它的介绍，请查看我们的[第一篇介绍 debuginfod](https://developers.redhat.com/blog/2019/10/14/introducing-debuginfod-the-elfutils-debuginfo-server/) 的文章和后续解释[如何建立自己的 debuginfod 服务](https://developers.redhat.com/blog/2019/12/17/deploying-debuginfod-servers-for-your-developers/)。

## 新 debuginfod 客户

Debuginfod 是 [elfutils](https://sourceware.org/elfutils/) 项目的一部分。已经使用 elfutils 来查找或分析调试资源的工具会自动继承`debuginfod`支持。像 [Systemtap](https://sourceware.org/systemtap/) 、 [Libabigail](https://sourceware.org/libabigail/) 和 [dwgrep](https://pmachata.github.io/dwgrep/) 这样的工具都是这样继承`debuginfod`的。例如，在 Systemtap 中，`debuginfod`提供了新的方法来指定要探测的进程。以前，如果您想浏览正在运行的用户进程，您必须提供进程标识符(PID)或可执行文件路径。使用`debuginfod`，Systemtap 也可以根据`build-id`探测进程。因此，可以独立于相应的可执行文件的位置来研究特定版本的二进制文件。

Debuginfod 包括一个客户端库(`libdebuginfod`)，它让其他工具可以轻松地查询`debuginfod`服务器上的源文件、可执行文件，当然还有`debuginfo`——通常是 DWARF(使用属性化记录格式进行调试)`debuginfo`。从去年开始，各种开发工具已经集成了`debuginfod`客户端。从版本 2.34 开始， [Binutils](https://www.gnu.org/software/binutils/) 包含了对使用单独`debuginfo` ( `readelf`和`objdump`)的组件的`debuginfod`支持。从 9.03 版本开始， [Annobin](https://developers.redhat.com/blog/2018/02/20/annobin-storing-information-binaries/) 项目包含了对获取单独`debuginfo`文件的`debuginfod`支持，并计划在 10.3 版本中支持 [Dyninst](https://dyninst.org/) 。

[GDB](https://www.gnu.org/software/gdb/) 10.1 最近发布，支持`debuginfod`，当你调试程序时，可以很容易地下载任何丢失的`debuginfo`或源文件，无论这些文件是用于被调试的可执行文件还是该可执行文件使用的任何共享库。GDB 还使用了对`libdebuginfod` API 的改进，包括可编程的进度更新，如下例所示(注意，为清晰起见，此输出被删节):

```
$ gdb /usr/bin/python
Reading symbols from /usr/bin/python...
Downloading separate debug info for /usr/bin/python
(gdb) list
Downloading source file /usr/src/debug/python3-3.8.6-1.fc32.x86_64/Programs/python.c...
8 wmain (int argc, wchar_t **argv)
9 {
10 return Py_Main(argc, argv)
11 }
(gdb) break main
Breakpoint 1 at 0x1140: file /usr/src/debug/python3-3.8.6-1.fc32.x86_64/Programs/python.c, line 16.
(gdb) run
Starting program: /usr/bin/python
Downloading separate debug info for /lib64/ld-linux-x86-64.so.2...
Downloading separate debug info for /lib64/libc.so.6...
Downloading separate debug info for /lib64/libpthread.so.0...
[...]

```

配置`debuginfod`为所有这些工具提供调试资源就像用`debuginfod`服务器的 URL 设置一个环境变量(`DEBUGINFOD_URLS`)一样简单。如果您不想设置自己的服务器，我们还提供了包含许多常见 Fedora、CentOS、Ubuntu、Debian 和 OpenSUSE 软件包的调试资源的服务器。更多信息，请浏览 [elfutils debuginfod 页面](https://sourceware.org/elfutils/Debuginfod.html)。

## 新的 debuginfod 服务器指标

为他人操作服务器是一种乐趣，也是一件苦差事。一旦你有了用户，他们就会期望服务持续运行。虽然`debuginfod`是一个简单的服务器，但它仍然需要监控和管理。考虑到这一点，`debuginfod`附带了通常的 log-to-stderr 标志，这些标志是为容器或 systemd 操作定制的。(另加`-v`了解更多信息。)此外，`debuginfod`还提供了一个网络应用编程接口，用于共享有关其内部运营的各种指标。这些指标在 [Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/) 中导出，这是行业标准的、人类可读的，并带有许多消费者和处理工具。这些指标旨在让您了解各个线程正在做什么，它们的工作负载进展如何，以及它们遇到了什么类型的错误。当在时间序列数据库中存档并进行简单分析时，这些指标可能会帮助您获得各种指导资源分配的整洁数量。

## 为 debuginfod 配置 Prometheus

要配置 Prometheus 服务器来收集`debuginfod`指标，请在`prometheus.yml`配置文件中添加一个 HTTP 或 HTTPS 子句，如下所示:

```
     scrape_configs:
       - job_name: 'debuginfod'
         scheme: http
         static_configs:
         - targets: ['localhost:8002']
       - job_name: 'debuginfod-https'
         scheme: https
         static_configs:
         - targets: ['debuginfod.elfutils.org'] # adjust

```

如果愿意，调整全局`scrape_interval`。Debuginfod 可以快速处理`/metrics`查询。让它运行一会儿，然后让我们浏览一下指标。

## 可视化 debuginfod 指标

当`debuginfod`第一次被指示扫描大目录的档案或文件时，它使用线程池(`-c option`)来解压缩和解析它们。这个活动可能是 I/O 和 CPU 密集型的，最好是两者都是！我们怎么知道？看看 *scanned_bytes_total* 度量，它列出了处理的输入文件`debuginfod`的总大小。当转换为速率时，它接近源文件系统的读取吞吐量。

**注意**:下面的截图是由内置的普罗米修斯图形生成的，但是你也可以使用另一个可视化工具，比如 [Grafana](https://grafana.com/) 。

### 测量扫描的总字节数

图 1 中的图表代表了一个密集的扫描作业，其中一个远程 NFS 服务器在一段时间内以稳定的 50 兆字节的速度向`debuginfod`提供数据，随后是不太明显的 10 兆字节。我们认为周一的到来可能是扫描性能下降的原因。开发人员周末回来后，不得不分享 NFS 的产能。

[![The graph shows a sudden drop in scanning performance.](img/de9af7baf3920f4aae47e4faab921337.png "db-pm1")](/sites/default/files/blog/2020/11/db-pm1.png)

Figure 1: Results from debuginfod's scanned_bytes_total metric displayed in a Prometheus graph.

如您所见，初始扫描一直在进行。开发人员不断开发，但是 NFS 服务器运行得越来越慢。为了进行分析，我们可以看看 *thread_work_pending* 指标。

### 测量线程活动

每当周期性遍历开始时, *thread_work_pending* 度量就会跳跃(`-t`选项和`SIGUSR1`),并随着那些扫描器线程执行它们的工作而返回到零。图 2 中的图表显示了扫描一个数 TB[Red Hat Enterprise Linux 8](https://developers.redhat.com/products/rhel/overview)RPM 数据集的五天时间。缓坡期对应于几个具有巨大 RPM 大小和许多构建(内核、RT-内核、Ceph、LibreOffice)的独特组合的包。急剧上升和下降对应于并发的重新遍历，由于索引数据仍然是新的，因此立即被忽略。当直线接触零点时，扫描完成。之后，应该只显示短暂的脉冲。

[![This graph shows sharp upticks and downticks.](img/c67738edf4fd80fe66a00b5003bba0dd.png "db-pm2")](/sites/default/files/blog/2020/11/db-pm2.png)

Figure 2: Results from debuginfod's thread_work_pending metric displayed in a Prometheus graph.

甚至在所有扫描完成之前，服务器已经准备好回答查询。毕竟，这就是它的全部——让开发者享受`debuginfo`的甜蜜甘露。但是有多少人在使用它，成本是多少？让我们检查一下 *http_responses_total* 指标，它对 web API 请求进行计数和分类。

### 测量 HTTP 响应

图 3 中的图表显示了一个小的错误峰值(未知的`build-id`),大量的成功(提取内容。rpm)，以及极少数其他的成功(使用`fdcache`)。这是一个批量的、发行版范围的`debuginfod`扫描的工作负载，它不能利用任何重要的缓存或预取。

[![The graph shows a sharp incline and a gradual decline.](img/f69e2e9ec5a18f0d9bef18b35b147505.png "db-pm3")](/sites/default/files/blog/2020/11/db-pm3.png)

Figure 3: Results from debuginfod's http_responses_total metric displayed in a Prometheus graph.

我们也来看看成本吧。如果您通过网络数据按字节测量成本，请调出*http _ responses _ transfer _ bytes*度量对。如果用 CPU 时间来衡量成本，请调出*http _ responses _ duration _ milliseconds*度量对。使用 PromQL，您可以计算平均数据传输或处理时间。

### 测量处理时间、整理统计数据和错误计数

图 4 中的图表显示了图 3 中相同时间范围的持续时间变量。它揭示了无法缓存或预取结果有时需要几十秒钟的服务时间，这可能是因为扫描同样的大型归档需要很长时间。配置主动缓存有助于创建更典型的访问模式。参见提到`fdcache`的指标。

[![need alt text.](img/8d7e8a6981495d54425e9e03dcccaa6f.png "db-pm4")](/sites/default/files/blog/2020/11/db-pm4.png)

Figure 4: Measuring processing time with debuginfod metrics in Prometheus.

既然您的服务器已经启动，它还会定期整理它的索引(`-g`选项和`SIGUSR2`)。作为每个整理周期的一部分，另一组指标被更新以提供整个索引的概览。最后几个数字给出了一个相当大的安装的存储要求的概念:6.58TB 的 rpm，76.6GB 的索引数据:

```
        groom{statistic="archive d/e"} 11837375
        groom{statistic="archive sdef"} 152188513
        groom{statistic="archive sref"} 2636847754
        groom{statistic="buildids"} 11477232
        groom{statistic="file d/e"} 0
        groom{statistic="file s"} 0
        groom{statistic="filenames"} 163330844
        groom{statistic="files scanned (#)"} 579264
        groom{statistic="files scanned (mb)"} 6583193
        groom{statistic="index db size (mb)"} 76662

```

`error_count`指标跟踪来自`debuginfod`的各个子系统的错误。

在这里，您可以看到错误是如何按子系统和类型分类的。我们希望这些指标的增加可以用来表示逐渐退化或彻底失败。我们建议向它们附加警报。

```
        error_count{libc="Connection refused"}  3
        error_count{libc="No such file or directory"}   1
        error_count{libc="Permission denied"}   33
        error_count{libarchive="cannot extract file"}   1

```

最后，您可以使用 Grafana 来抓取`debuginfod` Prometheus 服务器，以准备信息丰富且时尚的仪表板，如图 5 所示。

[![The dashboard displays a variety of debuginfod metrics.](img/cb334c80514cfa2db06642731aeb6242.png "debuginfod-grafana")](/sites/default/files/blog/2020/11/debuginfod-grafana.png)

Figure 5: Debuginfod metrics displayed on a Grafana dashboard.

## 结论

本文概述了从`debuginfod`开始提供的新的客户端支持和指标。我们没有涵盖所有可用的指标，所以您可以自己检查一下。如果你想到更多有用的指标，请联系我们在 elfutils-devel@sourceware.org 的开发者。

*Last updated: February 23, 2021*
# 加速系统调用的 SystemTap 脚本监控

> 原文：<https://developers.redhat.com/blog/2019/04/03/speed-up-systemtap-script-monitoring-of-system-calls>

SystemTap 有大量称为 tapsets 的库，允许开发人员检测内核操作的各个方面。SystemTap 允许使用通配符来检测特定子系统中的多个位置。SystemTap 必须执行大量的工作来为每个被探测的地方创建工具。当对包含数百个条目的系统调用 tapset(`syscall.*`和`syscall.*.return`)使用通配符时，这种开销尤其明显。对于数据收集的某些子集，用`kernel.trace("sys_enter")`和`kernel.trace("sys_exit")`探针替换 SystemTap 脚本中通配符匹配的 syscall 探针将产生更小的插装模块，这些模块可以更快地编译和启动。在本文中，我将展示几个例子来说明这是如何工作的。

所有系统调用通过`kernel.trace("sys_enter")`探针进入内核，所有系统调用通过`kernel.trace("sys_exit")`探针退出内核。下面是 SystemTap 命令，列出了这些探测点和可用的参数。`kernel.trace("sys_enter")`探测器提供了`$id`变量，可以使用`syscall_name()`函数将其映射到系统调用名。`kernel.trace("sys_exit")`探针提供指示系统调用成功或失败的返回值`$ret`。

> ```
> $ stap -L 'kernel.trace("sys_*")'
> kernel.trace("raw_syscalls:sys_enter") $regs:struct pt_regs* $id:long int
> kernel.trace("raw_syscalls:sys_exit") $regs:struct pt_regs* $ret:long int
> ```

作为一个具体的例子，我们来看看 systemtap-client-3.3-3 . el7 . x86 _ 64 中包含的`syscalls_by_pid.stp`例子。它使用系统调用工具的非 dwarf 变体:`nd_syscall.*`。

> ```
> global syscalls
> 
> probe begin {
> print ("Collecting data... Type Ctrl-C to exit and display results\n")
> }
> 
> probe nd_syscall.* {
> syscalls[pid()]++
> }
> 
> probe end {
> printf ("%-10s %-s\n", "#SysCalls", "PID")
> foreach (pid in syscalls-)
> printf("%-10d %-d\n", syscalls[pid], pid)
> }
> ```

下面是原始脚本的运行。`-v`选项提供了 SystemTap 使用的五个通道的空间和时间信息。`-k`选项保留了 SystemTap 创建的中间文件，因此如果需要的话，稍后我们可以将这些中间文件的大小与修改后的脚本的结果进行比较。`--disable-cache`确保 SystemTap 不会试图使用文件的先前构建的版本并破坏时间测量。`-m slow`将生成的 SystemTap 工具模块命名为 slow.ko。

命令的末尾是`-c "sleep 0"`。这个命令在很短的时间内运行脚本，允许我们观察脚本第 5 阶段的启动和关闭开销。

> ```
> $ stap -v -k --disable-cache -m slow ./syscalls_by_pid.stp -c "sleep 0"
> Pass 1: parsed user script and 497 library scripts using 264396virt/59208res/3572shr/56344data kb, in 390usr/30sys/423real ms.
> Pass 2: analyzed script: 531 probes, 27 functions, 103 embeds, 4 globals using 391992virt/187976res/4716shr/183940data kb, in 8170usr/1370sys/9276real ms.
> Pass 3: translated to C into "/tmp/stapos95gX/slow_src.c" using 391992virt/188212res/4952shr/183940data kb, in 40usr/0sys/45real ms.
> Pass 4: compiled C into "slow.ko" in 8350usr/1660sys/9794real ms.
> Pass 5: starting run.
> Collecting data... Type Ctrl-C to exit and display results
> #SysCalls  PID
> 42         19096
> 10         19078
> Pass 5: run completed in 10usr/9450sys/9940real ms.
> Keeping temporary directory "/tmp/stapos95gX"
> ```

下面是脚本的新版本。请注意，唯一的区别是将`nd_syscall.*`改为`kernel.trace("sys_enter")`。该脚本的运行方式与之前相同。

> ```
> global syscalls
> 
> probe begin {
> print ("Collecting data... Type Ctrl-C to exit and display results\n")
> }
> 
> probe kernel.trace("sys_enter") {
> syscalls[pid()]++
> }
> 
> probe end {
> printf ("%-10s %-s\n", "#SysCalls", "PID")
> foreach (pid in syscalls-)
> printf("%-10d %-d\n", syscalls[pid], pid)
> }
> 
> ```

下面是新脚本的运行。使用`kernel.trace("sys_enter")`代替`nd_syscall.*`的一个值得注意的效果是，在第二阶段，即 SystemTap 从 tapset 库获取信息的细化阶段，只使用了 3 个探针、1 个函数和 3 个嵌入，相比之下，通配符 syscall 版本使用了 531 个探针、27 个函数和 103 个嵌入。这一变化对脚本实际运行的阶段 5 有后续影响。因为脚本运行的持续时间非常短，所以 pass 5 中的大部分时间都是由于启动和停止脚本的开销。对于使用`nd_syscall.*`来探测系统调用的早期脚本，在 pass 5 中使用了 9940 毫秒，几乎 10 秒的实时，但是对于 tracepoint 版本，仅使用了 363 毫秒的实时。

> ```
> $ stap -v -k --disable-cache -m fast ./syscalls_by_pid.stp -c "sleep 0"
> Pass 1: parsed user script and 497 library scripts using 264400virt/59208res/3572shr/56348data kb, in 410usr/30sys/456real ms.
> Pass 2: analyzed script: 3 probes, 1 function, 3 embeds, 1 global using 389600virt/185704res/4856shr/181548data kb, in 45190usr/8990sys/9842real ms.
> Pass 3: translated to C into "/tmp/stapPcXVCr/fast_src.c" using 389600virt/185948res/5100shr/181548data kb, in 0usr/10sys/1real ms.
> Pass 4: compiled C into "fast.ko" in 6900usr/1730sys/8231real ms.
> Pass 5: starting run.
> Collecting data... Type Ctrl-C to exit and display results
> #SysCalls  PID
> 42         11076
> 14         6381
> 11         2715
> 10         11071
> Pass 5: run completed in 20usr/50sys/363real ms.
> Keeping temporary directory "/tmp/stapPcXVCr"
> 
> ```

跟踪点版本的最终检测也比通配符版本小得多，分别为 97KB 和 425KB:

> ```
> $ ls -l */*.ko
> -rw-rw-r--. 1 wcohen wcohen 97392 Mar 18 15:32 fast/fast.ko
> -rw-rw-r--. 1 wcohen wcohen 425408 Mar 18 15:25 slow/slow.ko
> ```

前面的例子没有通过名字跟踪系统调用。系统调用名可以从`kernel.trace("sys_enter")`的`$id`变量中获得。下面是一个单行脚本，它将记录每个不同的系统调用在系统上使用的次数:

> ```
> stap -e 'global syscalls_count; probe kernel.trace("sys_enter") {syscalls_count[syscall_name($id)]<<<1}'
> ```

由于在脚本中使用这些跟踪点的好处，SystemTap 4.0 包括了[syscall _ any tapset](https://sourceware.org/systemtap/tapsets/syscall_any.stp.html)和各种 [SystemTap 示例](https://sourceware.org/systemtap/examples/)现在尽可能使用 tap set。您应该考虑使用`kernel.trace("sys_enter")`和`kernel.trace("sys_exit")`或者`syscall_any` tapset 对您的脚本进行类似的优化，以监控所有的系统调用。

另请参见:

*   [使用 syscall_any tapset 减少 SystemTap 监控脚本的启动开销](https://developers.redhat.com/blog/2018/11/08/systemtap-reduced-startup-syscalls/)

*Last updated: May 1, 2019*
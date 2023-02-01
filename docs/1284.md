# 介绍 STAP pbf-SystemTap 的新 bpf 后端

> 原文：<https://developers.redhat.com/blog/2017/12/13/introducing-stapbpf-systemtaps-new-bpf-backend>

SystemTap 3.2 包括 SystemTap 的新 BPF 后端(stapbpf)的早期原型。它代表了利用最近添加到 Linux 内核中的强大的新跟踪和性能分析功能的第一步。在这篇文章中，我将比较 stapbpf 和默认后端(stap)的翻译过程，并比较这两个后端在功能上的一些差异。

Stap 和 stapbpf 共享共同的解析和语义分析阶段。作为翻译的输入，两个后端都接收表示解析树的数据结构，该解析树包含类型信息和对所使用的所有变量和函数的定义的引用。使用 stap 命令的'-p2 '选项可以显示该信息的摘要。

```
$ cat sample.stp
probe kernel.function("sys_read") { printf("hi from sys_read!\n"); exit() }

$ stap -p2 sample.stp
# functions
exit:unknown ()
kernel.function("SyS_read@fs/read_write.c:542") /* pc=_stext+0x273da0 */ /* <- kernel.function("SyS_read@fs/read_write.c:542") */

$ stap -p2 --runtime=bpf sample.stp
# functions
_set_exit_status:long ()
exit:unknown ()
# probes
kernel.function("SyS_read@fs/read_write.c:542") /* pc=_stext+0x273da0 */ /* <- kernel.function("SyS_read@fs/read_write.c:542") */

```

您可以看到 stapbpf 的 exit 函数包含了对`_set_exit_status`的额外调用，但除此之外，两个后端都在探测相同的位置。

从这一点出发，翻译过程出现分歧。Stap 的目标是将脚本转换成内核模块。为此，stap 将解析树翻译成所需内核模块的 C 源代码。在运行时，GCC 用于将这个源代码编译成实际的内核模块。'-p4 '选项可以与 stap 命令一起使用，以生成内核目标文件。

```
# stap -p4 sample.stp
[...]_1316.ko
# staprun [...]_1316.ko
hi from sys_read!

```

代替 C 语言，stapbpf 将脚本直接翻译成 bpf 字节码，由内核虚拟机执行。然后字节码被存储在一个供 stapbpf 运行时使用的 BPF-ELF 文件中。

```
# stap -p4 --runtime=bpf sample.stp
stap_1348.bo
# stapbpf stap_1348.bo
hi from sys_read!

```

与 stap 的内核模块不同，生成 BPF 字节码不需要外部编译器。这有助于降低 stapbpf 的编译时间和安装占用空间。使用'-v '选项，我们可以看到每个翻译阶段的持续时间。

```
# stap -v -p4 sample.stp
[...]
Pass 3: translated to C [...] in 0usr/0sys/4real ms.
Pass 4: compiled C [...] in 1330usr/310sys/1559real ms.

# stap -v -p4 --runtime=bpf sample.stp
[...]
Pass 4: compiled BPF into "stap_3792.bo" in 0usr/0sys/0real ms.

```

请注意，stap 的第 3 次和第 4 次传递需要 1563 毫秒，但 stapbpf 需要< 1 毫秒(stapbpf 将第 3 次和第 4 次传递合并为一次传递)。

当将 BPF 程序加载到内核时，它们首先由内核内置的 BPF 验证器进行安全性检查。它检查不受欢迎的行为，例如越界跳转、越界堆栈加载/存储和从未初始化的地址读取。它还检查是否存在无法到达的指令。任何没有通过验证的 BPF 程序都不会被加载到 BPF 虚拟机中。尽管默认的 stap 具有相似的标准，并且使用起来非常安全，但是 stapbpf 的优势在于继承了 bpf 更简单的安全模型。

然而，这种优势也伴随着一些代价。例如，BPF 不支持写入内核内存。尽管 stap 在默认情况下禁用了这一功能，但它确实提供了一种“guru 模式”,为希望对其操作系统进行这种级别控制的用户提供了一个逃生出口。这意味着 stapbpf 不具备 stap 的能力，例如，为实时系统管理安全创可贴。更具限制性的是，为了确保 BPF 计划迅速终止；验证者拒绝任何有循环的程序。虽然 stapbpf 可以执行循环展开，但 bpf 也对每个程序施加了 4096 条指令的限制。

```
# stap --runtime=bpf contains_loops.stp
Error loading /tmp/stapxSM7Kg/stap_8316.bo: bpf program load failed: Invalid argument
[...]
Pass 5: run failed.

# stap --runtime=bpf too_many_insns.stp
Error loading /tmp/stapqxRXi4/stap_8432.bo: bpf program load failed: Argument list too long
[...]
Pass 5: run failed.

```

下表是 stap 和 stapbpf 的比较总结。BPF 允许但尚未在 stapbpf 中实施的功能用“可能”表示。

|  | *步骤* | *stapbpf* |
| 非阻塞探测器处理程序 | 是 | 是 |
| 受保护的探测器执行环境 | 是 | 是 |
| 锁保护的全局变量 | 每个探针锁定 | 每次操作锁定 |
| kprobes(矮人) | 是 | 是 |
| kprobes(无矮人) | 是 | 可能的 |
| 向上倾斜 | 是 | 可能的 |
| 跟踪点探针 | 是 | 可能的 |
| 探测动态加载的内核对象 | 是 | 可能的 |
| 基于计时器的探测 | 是 | 是 |
| 能够改变被探测程序的状态 | 是 | 可能(仅用户空间) |
| 高级用户可以绕过保护的方法 | 是 | 不 |
| 循环支持(for，while，for each) | 是 | 有限* |
| 字符串支持(变量、文字) | 是 | 限量** |
| 探针处理器长度限制 | 1000 条语句 | 4096 指令 |
| 可用于增加处理器长度限制的方法 | 是 | 不 |
| 内核验证程序的安全性 | 不 | 是 |

*在开始和结束探测中启用 For 和 while 循环。这些探测在用户空间中执行，因此不需要验证。
**支持`printf`的格式字符串文字。

可以看出，stapbpf 只能提供 stap 功能的一个子集。然而，对于其安全策略要么阻止完整内核模块后端，要么要求软件具有比 stap 更简单的安全模型的系统，stapbpf 旨在提供一种方便的方式来使用这个子集。

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: December 12, 2017*
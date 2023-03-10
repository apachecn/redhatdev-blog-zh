# 用 SystemTap 使代码的操作更加透明和明显

> 原文：<https://developers.redhat.com/blog/2018/05/14/making-the-operation-of-code-more-transparent-and-obvious>

您可以研究源代码并手动插入函数，如在["使用动态跟踪工具，Luke"](https://developers.redhat.com/blog/2018/05/11/use-the-dynamic-tracing-tools-luke/) 博客文章中所述，但是为什么不通过向应用程序代码添加用户空间标记来更容易地找到软件中的关键点呢？用户空间标记在 Linux 中已经存在很长时间了(从 2009 年开始)。非活动用户空间标记不会显著降低代码速度。当意外问题发生时，让它们可用可以让您更准确地了解软件内部正在做什么。使用用户空间标记，诊断工具可以更加便于移植，因为工具不需要依赖于工具源代码中的特定函数名或行号。测量点的命名还可以更清楚地说明什么事件与特定的测量点相关联。

例如，Red Hat Enterprise Linux 7 上的 Ruby MRI 有许多不同的插装点，作为 SystemTap tapset 提供。如果系统上安装了 SystemTap，如[所述什么是 SystemTap，如何使用？](https://access.redhat.com/solutions/5441)，可以用下面显示的`stap -L`命令列出已安装的 Ruby MRI 仪器点。这些事件显示了 Ruby 运行时中各种操作的开始和结束，比如垃圾收集(GC)标记和清扫的开始和结束。

```
$ stap -L "ruby.**"
ruby.array.create size:long file:string line:long $arg1:long $arg2:long $arg3:long
ruby.cmethod.entry classname:string methodname:string file:string line:long $arg1:long $arg2:long $arg3:long $arg4:long
ruby.cmethod.return classname:string methodname:string file:string line:long $arg1:long $arg2:long $arg3:long $arg4:long
ruby.find.require.entry requiredfile:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.find.require.return requiredfile:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.gc.mark.begin
ruby.gc.mark.end
ruby.gc.sweep.begin
ruby.gc.sweep.end
ruby.hash.create size:long file:string line:long $arg1:long $arg2:long $arg3:long
ruby.load.entry loadedfile:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.load.return loadedfile:string $arg1:long $arg2:long $arg3:long
ruby.method.entry classname:string methodname:string file:string line:long $arg1:long $arg2:long $arg3:long $arg4:long
ruby.method.return classname:string methodname:string file:string line:long $arg1:long $arg2:long $arg3:long $arg4:long
ruby.object.create classname:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.parse.begin parsedfile:string parsedline:long $arg1:long $arg2:long
ruby.parse.end parsedfile:string parsedline:long $arg1:long $arg2:long
ruby.raise classname:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.require.entry requiredfile:string file:string line:long $arg1:long $arg2:long $arg3:long
ruby.require.return requiredfile:string $arg1:long $arg2:long $arg3:long
ruby.string.create size:long file:string line:long $arg1:long $arg2:long $arg3:long 
```

Ruby 和其他语言提供了一些工具来给出关于它们操作的一些方面的信息，比如关于垃圾收集的统计。然而，这些工具通常被设计成只在该语言和单个过程的范围内工作。这些特定于语言的工具对于检查跨越用不同语言编写的代码的问题或者多个通信进程之间的问题不太有用。现有的编译 C 库已经被插入到 Python、Ruby 和 Java 代码中。让 SystemTap 为这些共享库中的操作和事件设置 tapsets 可以更容易地编写诊断程序，从而更清楚地了解复杂系统的多个软件组件之间发生了什么。

例如，您可能有一个 Ruby 程序，它使用基于 GNOME GLib 库构建的 GUI。在进行屏幕更新时，您会观察到暂停。您怀疑暂停可能是由于 Ruby 的垃圾收集造成的，但是也认为暂停可能是由于 GLib 库操作中的问题造成的。GLib 库的开发人员已经在库中包含了许多插装点。在 Ruby 运行时和 GLib 库中都有插装点，这允许您使用一个工具 SystemTap 来检查这两个不同的代码区域。

随着时间的推移，各种软件包的开发人员和维护人员已经在他们的应用程序中添加了用户空间标记。在 Red Hat，我们已经在 RHEL 软件包中尽可能地启用了该工具。在 RHEL 7.5 上，许多 rpm 使得 SystemTap 用户空间探测器可用，如在`/usr/share/systemtap/tapset`中的文件查询所示:

```
$ cd /usr/share/systemtap/tapset; find -path "*.stp" -exec rpm -qf {} \; |sort |uniq
glib2-devel-2.54.2-2.el7.x86_64
java-1.7.0-openjdk-devel-1.7.0.171-2.6.13.2.el7.x86_64
libvirt-client-3.9.0-14.el7_5.2.x86_64
nodejs-6.14.0-1.el7.x86_64
pcp-3.12.2-5.el7.x86_64
perl-devel-5.16.3-292.el7.x86_64
python-libs-2.7.5-68.el7.x86_64
qemu-kvm-1.5.3-156.el7.x86_64
qemu-system-alpha-2.0.0-1.el7.6.x86_64
qemu-system-arm-2.0.0-1.el7.6.x86_64
qemu-system-cris-2.0.0-1.el7.6.x86_64
qemu-system-lm32-2.0.0-1.el7.6.x86_64
qemu-system-m68k-2.0.0-1.el7.6.x86_64
qemu-system-microblaze-2.0.0-1.el7.6.x86_64
qemu-system-mips-2.0.0-1.el7.6.x86_64
qemu-system-moxie-2.0.0-1.el7.6.x86_64
qemu-system-or32-2.0.0-1.el7.6.x86_64
qemu-system-s390x-2.0.0-1.el7.6.x86_64
qemu-system-sh4-2.0.0-1.el7.6.x86_64
qemu-system-unicore32-2.0.0-1.el7.6.x86_64
qemu-system-x86-2.0.0-1.el7.6.x86_64
qemu-system-xtensa-2.0.0-1.el7.6.x86_64
qemu-user-2.0.0-1.el7.6.x86_64
ruby-libs-2.0.0.648-33.el7_4.x86_64
sssd-common-1.16.0-19.el7.x86_64
systemtap-client-3.2-4.el7.x86_64
systemtap-devel-3.2-4.el7.x86_64
```

您应该考虑在您维护的应用程序中添加类似的、方便的探测点，以便您和其他人更容易调查复杂系统中的问题。[向应用程序添加用户空间探测(heapsort 示例)](https://sourceware.org/systemtap/wiki/AddingUserSpaceProbingToApps)提供了关于如何实现这种工具的信息。

*Last updated: May 29, 2018*
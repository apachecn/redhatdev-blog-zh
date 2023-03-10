# Open vSwitch DPDK PMD 线程核心关联性故障排除

> 原文：<https://developers.redhat.com/blog/2018/06/20/troubleshooting-open-vswitch-dpdk-pmd-thread-core-affinity>

当人们试图部署带有数据平面开发套件 (OvS-DPDK)的 [Open vSwitch 解决方案时，最常见的问题是性能不如预期。例如，他们正在丢失数据包。这是我们这一系列博客旅程的起点。](http://docs.openvswitch.org/en/latest/intro/install/dpdk/)

第一篇博客是关于轮询模式驱动(PMD)线程核心亲和力的。它涵盖了如何配置线程关联性，以及如何验证其设置是否正确。这包括确保没有其他线程正在使用 CPU 内核。

## 将 CPU 内核专用于 PMD 线程

PMD 线程是处理来自指定接收队列的数据包的接收和处理的线程。它们在一个紧密的循环中完成这项工作，任何中断这些线程的事情都会导致数据包被丢弃。这就是为什么这些线程**必须**运行在专用的 CPU 内核上；也就是说，系统中没有其他线程应该在这个内核上运行。对于各种 Linux 内核任务也是如此。

让我们假设您希望将 CPU 内核 1 和 15(单个超线程对)用于您的 PMD 线程。这将转换成`0x8002`的`pmd-cpu-mask`蒙版。

要手动完成隔离，您必须执行以下操作:

*   使用 Linux 内核命令行选项`isolcpus`将 PMD 内核与通用 SMP 平衡和调度算法隔离开来。对于上面的例子，您可以使用下面的:`isolcpus=1,15`。请注意，`isolcpus=`参数被弃用，取而代之的是 cpusets。更多信息请查看[内核文档](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt)。
*   使用组合的`nohz=on nohz_full=1,15`命令行选项可以减少时钟节拍中断的数量。这减少了 PMD 线程因服务定时器中断而被中断的次数。关于这个主题的更多细节可以在这里找到: [NO_HZ.txt](https://www.kernel.org/doc/Documentation/timers/NO_HZ.txt)
*   为了让上面的工作正常，我们需要另一个命令行选项，`rcu_nocbs=1,15`，否则内核仍然会中断线程；详情在同一个文档: [NO_HZ.txt](https://www.kernel.org/doc/Documentation/timers/NO_HZ.txt) 。

**注意**:对于上述内核选项，您可能需要添加额外的内核，这些内核也需要隔离。例如，分配给一个或多个虚拟机的核心和由`dpdk-lcore-mask`配置的核心。

为了使以上所有操作更加方便，您可以为此使用一个名为`cpu-partitioning`的调优概要文件。在 [tuned](https://servicesblog.redhat.com/2012/04/16/tuning-your-system-with-tuned/) 上有一个稍微老一点的博客可能会有帮助。然而，简而言之，这就是你如何配置它:

```
# systemctl enable tuned
# systemctl start tuned
# echo isolated_cores=1,15 >> /etc/tuned/cpu-partitioning-variables.conf
# echo no_balance_cores=1,15 >> /etc/tuned/cpu-partitioning-variables.conf
# tuned-adm profile cpu-partitioning
# reboot
```

**注意**:您仍然需要如上所述手动设置`isolcpus`，以实现零损耗性能。

## 验证 CPU 分配

### 命令行选项

首先，检查 Linux 命令行选项，看看它们是否按照预期进行了配置:

```
# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-693.17.1.el7.x86_64 \
root=/dev/mapper/rhel_wsfd--netdev64-root ro crashkernel=auto \
rd.lvm.lv=rhel_wsfd-netdev64/root rd.lvm.lv=rhel_wsfd-netdev64/swap \
console=ttyS1,115200 iommu=pt intel_iommu=on \
default_hugepagesz=1G hugepagesz=1G hugepages=32 \
isolcpus=1,2,3,4,5,6,15,16,17,18,19,20 skew_tick=1 \
nohz=on nohz_full=1,2,3,4,5,6,15,16,17,18,19,20 \
rcu_nocbs=1,2,3,4,5,6,15,16,17,18,19,20
tuned.non_isolcpus=0fe07f81 intel_pstate=disable nosoftlockup

```

### PMD 线亲和力

其次，确保 PMD 线程正在/将要在正确的线程上运行。在本例中，您可以看到它们被分配给了错误的 CPU 11 和 27:

```
# pidstat -t -p `pidof ovs-vswitchd` 1 | grep -E pmd\|%CPU
06:41:21      UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
06:41:22      995         -      1316  100.00    0.00    0.00  100.00    27  |__pmd33
06:41:22      995         -      1317  100.00    0.00    0.00  100.00    11  |__pmd32
06:41:22      UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
06:41:23      995         -      1316  100.00    0.00    0.00  100.00    27  |__pmd33
06:41:23      995         -      1317  100.00    0.00    0.00  100.00    11  |__pmd32
```

在这种情况下，这是由于`tuned`中的一个已知错误，它在初始化之前移走了运行在隔离内核上的进程。运行`sytemctl restart openvswitch`将解决这个具体问题:

```
# systemctl restart openvswitch
# pidstat -t -p `pidof ovs-vswitchd` 1 | grep -E pmd\|%CPU
06:44:01      UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
06:44:02      995         -      2774  100.00    0.00    0.00  100.00     1  |__pmd32
06:44:02      995         -      2775  100.00    0.00    0.00  100.00    15  |__pmd33
06:44:02      UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
06:44:03      995         -      2774  100.00    0.00    0.00  100.00     1  |__pmd32
06:44:03      995         -      2775  100.00    0.00    0.00  100.00    15  |__pmd33
```

**注意**:在`tuned`问题解决之前，你总是需要在`tuned`重启之后重启 OVS 来获得正确的 CPU 分配！

要 100%确定 Linux 内核没有在另一个内核上调度您的 PMD 线程，请使用`taskset`命令:

```
# taskset -pc 2774
pid 2774's current affinity list: 1
# taskset -pc 2775
pid 2775's current affinity list: 15
```

### 使用 PMD 内核的其他线程

最后，确保没有其他线程被调度到 PMD 内核上。以下命令将为所有正在运行的用户空间线程提供 CPU 关联:

```
find -L /proc/[0-9]*/exe ! -type l | cut -d / -f3 | \
  xargs -l -i sh -c 'ps -p {} -o comm=; taskset -acp {}'
```

下面是部分示例输出。这里您可以看到我的`bash`进程正在使用 PMD 保留的 CPU 内核:

```
...
agetty
pid 1443's current affinity list: 0,7-14,21-27
bash
pid 14863's current affinity list: 0-15
systemd
pid 1's current affinity list: 0,7-14,21-27
ovs-vswitchd
pid 3777's current affinity list: 2
pid 3778's current affinity list: 0,7-14,21-27
pid 3780's current affinity list: 2
pid 3781's current affinity list: 16
pid 3782's current affinity list: 2
pid 3785's current affinity list: 2
pid 3786's current affinity list: 2
pid 3815's current affinity list: 1
pid 3816's current affinity list: 15
pid 3817's current affinity list: 2
pid 3818's current affinity list: 2
pid 3819's current affinity list: 2
pid 3820's current affinity list: 2
...
```

**注意**:你也可以使用一个叫做 Tuna 的工具来列出在一个特定内核上运行的所有进程。

如果您验证了以上所有内容，就可以防止其他线程和内核干扰 PMD 线程。如果您对环境进行了任何重大更改，请不要忘记重新检查上述内容。

*Last updated: March 23, 2022*
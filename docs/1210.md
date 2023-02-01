# RHEL 无根开放式虚拟交换机

> 原文：<https://developers.redhat.com/blog/2018/03/23/non-root-open-vswitch-rhel>

几周后，快速数据路径生产通道将把 [Open vSwitch](https://www.openvswitch.org/) 版本从 2.7 系列更新到 2.9 系列。这在很多方面都是一个重要的变化。大量与数据包移动相关的新功能和修复将开始发挥作用。一个肯定会导致所有问题的原因是'- ovs-user '标志的集成，该标志允许非特权用户与 Open vSwitch 进行交互。

以 root 身份运行可以解决很多讨厌的问题。想要写入任意文件？没问题。想加载内核模块？去吧！想嗅探网络上的数据包吗？进行数据包转储。当控制计算机的人是合法的拥有者时，所有这些都是伟大的。但是当键盘前的人不是合法的主人时，问题就出现了。

可能有一个精明的读者已经提出了一些问题，为什么还要费心向非根用户锁定 OvS 二进制文件。毕竟，OvS 交换机使用 netlink 告诉内核移动端口，瞧！它发生了！这一点不会改变。但是，那是意料之中的。

另一方面，尽可能地限制 Open vSwitch 是有好处的。例如，Open vSwitch 不需要拥有允许向`/bin`写入新二进制文件的特权。此外，Open vSwitch 应该永远不需要访问 Qemu 磁盘文件。这些限制有助于将开路电压开关限制在较小的影响范围内。

自从 Open vSwitch 版以来，基础架构已经可以作为非 root 用户运行，但打开它似乎总是有点可怕。有人担心与 Qemu、libvirt 和 DPDK 的相互作用。更进一步，selinux 真的会出现问题。为了解决这些问题，已经进行了大量的后台工作，在 Fedora 中以这种方式运行了一段时间后，我们认为我们已经解决了最严重的问题。

那么，您需要做些什么来确保您的 Open vSwitch 实例作为非 root 用户运行呢？理想情况下什么都没有；全新安装的`openvswitch` rpm 将自动确保一切配置正确，可以作为非 root 用户运行。用`ps`检查时，这一点很明显:

```
$ ps aux | grep ovs
 openvsw+ 15169 0.0 0.0 52968 2668 ? S<s 10:30 0:00 ovsdb-s
 openvsw+ 15214 200 0.3 5840636 229332 ? S<Lsl 10:30 809:16 ovs-vs
```

对于新的安装，这应该足够了。当使用基于 vfio 的 PMD 时，甚至 DPDK 设备也可以工作(大多数 PMD 支持 vfio，因此您确实应该使用它)。

升级 Open vSwitch 版本的用户可能会发现 Open vSwitch 实例以 root 用户身份运行。这是故意的；我们不想打破任何现有的设置。然而，所有花哨的基础设施都在那里，如果你愿意，就可以进行转换。只需采取几个简单的步骤:

1.  编辑`**/etc/sysconfig/openvswitch**`并将`OVS_USER_ID`变量修改为`openvswitch:hugetlbfs`(或者您想要的任何用户)
2.  确保目录(`**/etc/openvswitch**`、`**/var/log/openvswitch**`和`**/dev/vfio**`)拥有正确的所有权/权限。这包括文件和子目录。
3.  启动守护进程(`systemctl start openvswitch`)。

如果这一步出错，通常会在`journalctl`(例如使用`journalctl -xe -u ovsdb-server`)或日志文件中显示出来。

一旦非根更改生效，您仍然可能会遇到一些权限问题，这些问题在 journalctl 中并不明显。最常见的是在使用 libvirtd 启动虚拟机时。在这种情况下，默认的 libvirt 配置(或者是 *Group=root* ，或者是 *Group=qemu* )可能不会授予正确的 groupid 来访问 vhost-user 套接字。这可以通过编辑`**/etc/libvirt/qemu.conf**`配置文件中的`Group=`设置进行配置，以匹配 Open vSwitch 的组(同样，默认为`hugetlbfs`)。

我希望这是有帮助的！

*Last updated: March 22, 2018*
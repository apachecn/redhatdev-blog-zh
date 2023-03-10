# 打开没有过时端口的 vSwitch

> 原文：<https://developers.redhat.com/blog/2017/12/01/open-vswitch-without-stale-ports>

Open vSwitch 每天都在增长，并被用于大规模部署。通常，这意味着在 vswitch 中配置的始终可用的端口很少，例如物理以太网端口和其他几个为虚拟机或容器提供网络连接的端口。这些其他端口是软件设备，例如在重启或系统崩溃后，它们通常不能被重用。

这篇博文将讨论如何确保 vSwitch 在系统崩溃或严重关机后恢复正常。这个想法是，一旦 vSwitch 启动，就不需要另一个组件(通常是一个远程控制器)来迭代大量过时的端口并清理它们。

## 它是如何工作的

上游接受了一个名为*“transient”*的新端口属性，预计在 OVS 2.9 中可用。出于向后兼容的原因，如果没有配置，默认为*【假】*，与之前的 OVS 版本没有任何行为变化。

然而，管理系统通常知道端口是否是软件设备。如果是，可以将*【瞬态】*属性设置为*【真】*。除了通知 Open vSwitch 服务初始化在服务启动之前从数据库中删除该端口之外，它不会做任何其他事情。

对于 Fedora 和 RHEL 发行版，有一个新的 systemd 服务负责清理。它名为*" ovs-delete-transient-ports . service "*，每次启动时运行一次，因此不会影响服务重启。该服务执行*【ovs-CTL】*脚本，通过*【删除-暂态-端口】*完成实际工作。

为了更深入地了解细节，systemd 进行服务编排。第一个被启动的服务是*“ovsdb-server”*。然后*“ovs-delete-transient-ports”*服务运行，该服务在数据库中查询带有*“transient = true”*的端口并删除它们。最后， *"ovs-vswitchd"* 服务启动，完成 vswitch 初始化。

我有一个脚本，如下所示，模拟连接到 Open vSwitch 的 1000 个容器，然后我将触发内核崩溃来模拟内核崩溃。在没有设置属性的情况下重新启动后，我们可以看到所有端口都保留在数据库中，但不在系统中。

create_containers.sh 的内容:

```
#!/bin/bash

# create a bridge
ovs-vsctl add-br br0

for ns in $(seq 1 $1)
do
    # add a new veth pair
    ip link add vethHOST${ns} type veth \
        peer name vethNS${ns}

    # add a new network namespace
    ip netns add ${ns}

    # add host port to the vswitch bridge
    ovs-vsctl add-port br0 vethHOST${ns}

    # moving veth peer to the namespace
    ip link set vethNS${ns} netns ${ns}

done
```

```
# ./create_containers.sh 1000
# # Checking the total number of software devices:
# ip link | grep vethHOST | wc -l
1000
# # Checking the total number of ports in br0:
# ovs-vsctl show | grep 'Port "vethHOST' | wc -l
1000
```

是时候触发系统崩溃来模拟故障了:

```
# echo b > /proc/sysrq-trigger
```

趁系统还没恢复，这是一个喝一杯你喜欢的饮料的好时机。

系统恢复了，让我们来验证软件设备是否幸存，以及它们是否在 OVS 大桥上。

```
# # Repeating the previous command checking the
# # number of software devices:
# ip link | grep vethHOST | wc -l
0
# # Repeating the previous command checking the
# # the number of ports in br0:
# ovs-vsctl show | grep 'Port "vethHOST' | wc -l
1000
```

尽管设备不存在，但端口仍保留在数据库中。现在，如果管理系统尝试添加与这 1000 个过时端口之一同名的新软件设备，它将收到一条错误消息:

```
# ovs-vsctl add-port ovsbr0 vethHOST114
ovs-vsctl: cannot create a port named vethHOST114
because a port named vethHOST114 already exists on
bridge br0
```

## 解决办法

让我们修改脚本，为所有这些软件设备设置*“transient = true”*，并重复测试。

```
#!/bin/bash

# create a bridge
ovs-vsctl add-br br0

for ns in $(seq 1 $1)
do
    # add a new veth pair
    ip link add vethHOST${ns} type veth \
        peer name vethNS${ns}

    # add a new network namespace
    ip netns add ${ns}

    # add host port to the vswitch bridge
    ovs-vsctl add-port br0 vethHOST${ns} \
    -- set Port vethHOST${ns} other_config:transient=true

    # moving veth peer to the namespace
    ip link set vethNS${ns} netns ${ns}

done
```

这是系统崩溃后的结果:

```
# # Repeating the previous command checking the
# # number of software devices:
# ip link | grep vethHOST | wc -l
0
# # Repeating the previous command checking the
# # the number of ports in br0:
# ovs-vsctl show | grep 'Port "vethHOST' | wc -l
0
```

br0 网桥上没有软件设备和端口。

## 结论

Open vSwitch 有一个数据库来以持久的方式存储配置，但有时这不是我们所希望的，尤其是对于 vnets、vhost-users 或 veth pairs 等软件设备，它们将无法在崩溃后存活。

Upstream 合并了临时端口布尔属性，以帮助在服务可用之前清理这些类型的端口。

谢谢！

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: November 30, 2017*
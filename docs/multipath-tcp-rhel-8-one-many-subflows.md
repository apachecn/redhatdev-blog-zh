# RHEL 8 上的多路径 TCP:从一个到多个子流

> 原文：<https://developers.redhat.com/articles/2021/10/20/multipath-tcp-rhel-8-one-many-subflows>

许多年前，赫拉克勒斯在返回雅典的途中，在森林里迷了路。到了一个十字路口，他发现了两个女人。其中一个是真理女神阿勒忒伊亚，另一个是谎言女神阿帕特。但是他不知道谁是谁。赫拉克勒斯需要他所有的智慧来寻找唯一能够揭示通往雅典之路的问题。

如果你迷失在一个计算机网络中，不知道走哪条路，不要担心——你不需要大力士来找到你的路:你可以使用多路径 TCP。

## Red Hat Enterprise Linux 8 中的多路径 TCP

多路径 TCP (MPTCP)是传输控制协议(TCP)的扩展，用于[在对等体之间同时使用多条路径](https://www.rfc-editor.org/info/rfc8684)。 [Linux](/topics/linux/) 的 MPTCP 实现是相当新的，包含在 5.6 版本中。 [Red Hat Enterprise Linux](/products/rhel) 从 8.3 版本开始包含 MPTCP。

在本系列第一部分的[中，](/blog/2020/08/19/multipath-tcp-on-red-hat-enterprise-linux-8-3-from-0-to-1-subflows)展示了如何启用 MPTCP，在应用程序中打开 MPTPC 套接字，并验证 MPTCP 是否按预期工作。您可以按照他的教程开始尝试使用 MPTCP。

在本文中，您将学习如何:

*   使用`iproute2`向 MPTCP 连接添加多个子流。
*   验证 MPTCP 使用多个子流。

## 打开 MPTCP 套接字

要查看运行中的 MPTCP，您需要从用户空间应用程序中打开一个 MPTCP 套接字。让我们按照本系列第一篇文章中的说明来建立一个多子流测试平台。

首先，因为 MPTCP 在默认的 Red Hat Enterprise Linux 配置中是禁用的，所以您需要使用`sysctl`来启用它，这样您就可以创建 MPTCP 套接字:

```
# sysctl -w net.mptcp.enabled=1
# sysctl net.mptcp.enabled
net.mptcp.enabled = 1
```

MPTCP 套接字与常规 TCP 套接字相同，并使用相同的语义。一个应用可以使用一个带有`IPPROTO_MPTCP`的套接字为 MPTCP 添加本地支持，如下所示:

```
fd = socket(AF_INET, SOCK_STREAM, IPPROTO_MPTCP);
```

实现一个成熟的 MPTCP 应用程序并不难，但是如果您想在没有任何`IPPROTO_MPTCP`知识的情况下使用一个普通的用户空间应用程序，该怎么办呢？

不用担心:有多种方法可以避免修补和重建所有网络应用程序。最简单的路径可能是在内核中所有对`__sys_socket()`的调用中，用`systemtap`将`IPPROTO_TCP`替换为`IPPROTO_MPTCP`。要进行替换，您需要安装几个软件包:

```
$  dnf -y install \
  kernel-headers \
  kernel-devel \
  kernel-debuginfo \
  kernel-debuginfo-common-x86_64 \
  systemtap-client \
  systemtap-devel
```

现在您可以从本指南中[下载`systemtap`脚本，并使用以下命令启动它:](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-multipath-tcp_configuring-and-managing-networking)

```
# stap -vg mpctp.stap
```

通过查看来自`dmesg`的输出，验证`systemtap`脚本正在运行:

```
# dmesg
...
[1114254.601040] Kprobes globally unoptimized
[1114254.611265] stap_d60b4bc4e0332aa247ebd9b2ffa597_213373: module_layout: kernel tainted.
[1114254.620296] Disabling lock debugging due to kernel taint
[1114254.626423] stap_d60b4bc4e0332aa247ebd9b2ffa597_213373: loading out-of-tree module taints kernel.
[1114254.636597] stap_d60b4bc4e0332aa247ebd9b2ffa597_213373: module verification failed: signature and/or required key missing - tainting kernel
[1114254.680368] stap_d60b4bc4e0332aa247ebd9b2ffa597_213373 (mptcp.stap): systemtap: 4.4/0.182, base: ffffffffc1082000, memory: 224data/32text/15ctx/24678net/202alloc kb, probes: 1
```

## 设置试验台

作为一个简单的测试设置，我们将使用两个网络名称空间`mptcp-client`和`mptcp-server`来模拟 MPTCP 连接中的客户机和服务器。这两个名称空间将通过两个不同的虚拟以太网(veth)路径连接:从 10.0.0.1 到 10.0.0.2 和从 192.168.0.1 到 192.168.0.2(图 1)。

[![Network topology for MPTCP two-stream test. In this setup, 10.0.0.1 on the server communicates with 10.0.0.2 on the client, and 192.168.0.1 on the server communicates with 192.168.0.2 on the client.](img/788673b67ae00089408f5340f76240c7.png)](/sites/default/files/setup.png)Figure 1: Network topology for two-path MPTCP test.

您可以使用以下脚本来设置测试平台:

```
#!/bin/sh

ip netns add mptcp-client
ip netns add mptcp-server

sysctl -w net.ipv4.conf.all.rp_filter=0
ip netns exec mptcp-client sysctl -w net.mptcp.enabled=1
ip netns exec mptcp-server sysctl -w net.mptcp.enabled=1

ip link add red-client netns mptcp-client type veth peer red-server netns mptcp-server
ip link add blue-client netns mptcp-client type veth peer blue-server netns mptcp-server

ip -n mptcp-server address add 10.0.0.1/24 dev red-server
ip -n mptcp-server address add 192.168.0.1/24 dev blue-server
ip -n mptcp-client address add 10.0.0.2/24 dev red-client
ip -n mptcp-client address add 192.168.0.2/24 dev blue-client

ip -n mptcp-server link set red-server up
ip -n mptcp-server link set blue-server up
ip -n mptcp-client link set red-client up
ip -n mptcp-client link set blue-client up
```

您可以通过遵循本系列第一部分中的说明[来验证您的设置是否按预期工作。](/blog/2020/08/19/multipath-tcp-on-red-hat-enterprise-linux-8-3-from-0-to-1-subflows)

## 使用多条路径

既然您已经让 MPTCP 在一条路径上工作，那么是时候采用多条路径了。

首先，指示内核设置多个 MPTCP 子流。`iproute2`提供了一个方便的`mptcp`命令，可以帮助您:

```
# ip -n mptcp-server mptcp endpoint flush
# ip -n mptcp-server mptcp limits set subflow 2 add_addr_accepted 2
# ip -n mptcp-client mptcp endpoint flush
# ip -n mptcp-client mptcp limits set subflow 2 add_addr_accepted 2
# ip -n mptcp-client mptcp endpoint add 192.168.0.2 dev blue-client id 1 subflow
```

这些命令将 MPTCP 服务器配置为最多接受两个不同的子流，然后向客户端添加第二个子流。再次使用`iproute2`验证所有配置是否符合预期:

```
# ip -n mptcp-server mptcp limit show
add_addr_accepted 2 subflows 2
# ip -n mptcp-client mptcp limit show
add_addr_accepted 2 subflows 2
# ip -n mptcp-client mptcp endpoint show
192.168.0.2 id 1 subflow dev blue-client

```

现在您已经准备好使用子流了。要测试它们，可以使用`ncat`。以下命令在`mptcp-server`上启动一个`ncat`服务器实例:

`# ip netns exec mptcp-server ncat -k -4 -i 30 -c "sleep 60" -C -o /tmp/server -l 0.0.0.0 4321 &`

接下来，下面的命令发送“hello world！”从`mptcp-client`名称空间到服务器的消息:

```
$ ip netns exec mptcp-client ncat -c "echo hello world!" 10.0.0.1 4321
```

使用`tcpdump`，您可以验证来自不同接口的两个不同的三向握手:

```
# tcpdump --number -tnnr /tmp/mptcp.pcap
reading from file /tmp/mptcp.pcap, link-type LINUX_SLL (Linux cooked v1)
dropped privs to tcpdump
1  IP 10.0.0.2.43474 > 10.0.0.1.4321: Flags [S], seq 908898843, win 29200, options [mss 1460,sackOK,TS val 3701631927 ecr 0,nop,wscale 7,mptcp capable[bad opt]>
2  IP 10.0.0.1.4321 > 10.0.0.2.43474: Flags [S.], seq 3314791626, ack 908898844, win 28960, options [mss 1460,sackOK,TS val 3198006599 ecr 3701631927,nop,wscale 7,mptcp capable Unknown Version (1)], length 0
3  IP 10.0.0.2.43474 > 10.0.0.1.4321: Flags [.], ack 1, win 229, options [nop,nop,TS val 3701631927 ecr 3198006599,mptcp capable Unknown Version (1)], length 0
4  IP 10.0.0.2.43474 > 10.0.0.1.4321: Flags [P.], seq 1:14, ack 1, win 229, options [nop,nop,TS val 3701631928 ecr 3198006599,mptcp capable[bad opt]>
5  IP 10.0.0.1.4321 > 10.0.0.2.43474: Flags [.], ack 14, win 227, options [nop,nop,TS val 3198006600 ecr 3701631928,mptcp dss ack 3158259848540329265], length 0
6  IP 192.168.0.2.36423 > 10.0.0.1.4321: Flags [S], seq 2791202022, win 29200, options [mss 1460,sackOK,TS val 1604001975 ecr 0,nop,wscale 7,mptcp join id 1 token 0xc0715389 nonce 0xcae83dcb], length 0
7  IP 10.0.0.1.4321 > 192.168.0.2.36423: Flags [S.], seq 637604674, ack 2791202023, win 28960, options [mss 1460,sackOK,TS val 511057212 ecr 1604001975,nop,wscale 7,mptcp join id 0 hmac 0x465e4bf08492bb0c nonce 0x47d18eca], length 0
8  IP 10.0.0.2.43474 > 10.0.0.1.4321: Flags [.], ack 1, win 229, options [nop,nop,TS val 3701631928 ecr 3198006600,mptcp dss fin ack 641236127 seq 3158259848540329265 subseq 0 len 1,nop,nop], length 0
9  IP 10.0.0.1.4321 > 10.0.0.2.43474: Flags [.], ack 14, win 227, options [nop,nop,TS val 3198006600 ecr 3701631928,mptcp dss ack 3158259848540329266], length 0
10  IP 10.0.0.2.43474 > 10.0.0.1.4321: Flags [F.], seq 14, ack 1, win 229, options [nop,nop,TS val 3701631928 ecr 3198006600,mptcp dss ack 641236127], length 0
11  IP 10.0.0.1.4321 > 10.0.0.2.43474: Flags [.], ack 15, win 227, options [nop,nop,TS val 3198006641 ecr 3701631928,mptcp dss ack 3158259848540329266], length 0
```

事实上，第一次三次握手发生在 10.0.0 上的数据包 1-3。*路径，而第二次握手则在 192.168.0 上的数据包 6-8 中开始。* path(注意 SYN 的 S 标志和 SYN/ACK 的 S.)。

## 厌倦了手动分流？mptcpd 前来救援

到目前为止，您可能已经厌倦了为您想要做的每个新测试手工指定端点和子流程。幸运的是，这不是使用 MPTCP 端点的唯一方式。

Linux 内核版本 5.11 能够在内核接收到`add address netlink`命令时向用户空间发送`netlink`通知。一个小的用户空间应用程序`mptpcd`可以利用这些通知来提供一个到用户空间的路径管理机制，并控制 MPTCP 行为。`mptpcd`将在 RHEL 9 中提供。

## 结论

最后，赫拉克勒斯设法找到了去雅典的路，他只是简单地问了一位女神另一位女神会告诉他走哪条路。他选择了另一条路，很快回到了雅典。

有了 MPTCP，你不再需要解决路径之谜:你可以询问多条道路，并把它们一起带到你需要去的地方。

*Last updated: October 6, 2022*
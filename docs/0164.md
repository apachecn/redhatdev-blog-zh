# 更简单的方法:Linux 内核中 UDP 上的 SCTP

> 原文：<https://developers.redhat.com/articles/2021/06/04/easier-way-go-sctp-over-udp-linux-kernel>

用户数据报协议上的流控制传输协议(UDP 上的 SCTP，也称为 SCTP 的 UDP 封装)是在 [RFC6951](https://datatracker.ietf.org/doc/html/rfc6951) 中定义的特性，并从 5.11.0 开始在 [Linux](/topics/linux) 内核空间中实现。计划由[红帽企业 Linux](/products/rhel/overview) (RHEL) 8.5.0 和 9.0 支持。

本文简要介绍了 Linux 内核中 UDP 上的 SCTP。

## 为什么我们需要 SCTP 而不是 UDP

正如互联网工程任务组征求意见稿(RFC)中所述，我们需要 SCTP 而不是 UDP 有两个主要原因:

```
To allow SCTP traffic to pass through legacy NATs, which do not
provide native SCTP support as specified in [BEHAVE] and
[NATSUPP].

To allow SCTP to be implemented on hosts that do not provide
direct access to the IP layer. In particular, applications can
use their own SCTP implementation if the operating system does not
provide one.
```

第一个原因将解决给用户带来许多麻烦并阻止 SCTP 广泛使用的中间体问题。第二个原因是允许用户空间应用程序基于 UDP 协议开发自己的 SCTP 实现。

## UDP 上的 SCTP 如何工作

启用此功能后，所有 SCTP 数据包都封装到 UDP 数据包中。UDP 上的 SCTP 是用内核 UDP 隧道 API 实现的，这些 API 以前被 VXLAN、GENEVE 和 TIPC 协议使用。

为了接收封装的数据包，内核监听所有本地接口上的一个特定 UDP 端口。默认端口是 9899。该端口还充当该主机作为发送方的 UDP 数据包`src`端口。如您所料，`dest`端口应该是对等体的监听端口，默认为 9899。SCTP 绑定到的`src`和`dest`地址仍将在 IP 报头中使用:

```
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    IP(v6) Header      (addresses bound by SCTP) |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    UDP Header         (src: 9899, dest: 9899)   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    SCTP Common Header (SCTP ports)              |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    SCTP Chunk #1                                |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    ...                                          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    SCTP Chunk #n                                |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
```

**注意** : SCTP 在计算碎片点时也会考虑 UDP 报头。

## 如何通过 UDP 使用 SCTP

编程时，您不需要做任何改变:所有标准的 SCTP 特性仍然适用，所有的 API 都可以像以前一样使用。旧的应用程序不需要任何修改或重新编译就可以很好地工作。唯一的调整是设置一个 UDP 端口(本地监听端口或`src`端口)和一个*封装*端口(远程监听或`dest`端口)，这可以由`sysctl`为网络名称空间全局完成:

```
 # sysctl -w net.sctp.encap_port=9899
    # sysctl -w net.sctp.udp_port=9899
```

或者，您可以使用`sockopt`为每个套接字、关联或传输设置封装端口:

```
 setsockopt(SCTP_REMOTE_UDP_ENCAPS_PORT, port);
```

在服务器端，`encapsulation`端口通常不需要显式设置，下一节将详细介绍。

## UDP 封装端口

UDP 封装端口允许非常灵活的使用。在发送端，全局封装端口仅提供默认值:

*   当一台主机上的另一个套接字连接到使用不同 UDP 端口的另一台主机时，可以使用*每套接字*封装端口。
*   当同一个套接字连接到使用不同 UDP 端口的不同主机时，可以使用*每关联*封装端口。
*   当同一个关联想要在一个传输上发送 UDP 封装的 SCTP 数据包时，可以使用*每传输*封装端口。

在接收端，封装端口通常不需要设置:

*   一个关联的封装端口将从第一个`INIT`分组中获知。具有不同 UDP `src`端口的其他`INIT`将被丢弃。
*   每个传输的封装端口将从相应路径上的传入分组中获知，并且可以随时更新。
*   即使设置了关联的封装端口及其传输，普通 SCTP 数据包仍然可以被处理。

## 结论

如果您正在使用 SCTP，并且喜欢它的特性，比如多宿主、多流和部分可靠性，但是有中间盒的问题，Linux 内核现在提供了一种更简单的方法来解决这些问题。

*Last updated: August 24, 2022*
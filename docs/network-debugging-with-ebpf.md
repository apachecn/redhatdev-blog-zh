# 使用 eBPF 进行网络调试(RHEL 8)

> 原文：<https://developers.redhat.com/blog/2018/12/03/network-debugging-with-ebpf>

## 介绍

与网络打交道很有趣，但通常也是麻烦的来源。网络故障排除可能很困难，重现现场发生的不良行为也很痛苦。

幸运的是，有一些工具可以提供帮助:网络名称空间、虚拟机、`tc`和`netfilter`。简单的网络设置可以用网络名称空间和`veth`设备来重现，而更复杂的设置需要用软件桥连接虚拟机，并使用标准的网络工具，如`iptables`或`tc`，来模拟不良行为。如果由于 SSH 服务器关闭而导致生成的 ICMP 回复有问题，那么在正确的名称空间或 VM 中的`iptables -A INPUT -p tcp --dport 22 -j REJECT --reject-with icmp-host-unreachable`可以解决这个问题。

本文描述了如何使用 [eBPF(扩展 BPF)](https://lwn.net/Articles/740157/)(Berkeley 数据包过滤器的扩展版本)来解决复杂的网络问题。eBPF 是一项相当新的技术，该项目仍处于早期阶段，文档和 SDK 尚未准备好。但这应该会有所改善，特别是随着 XDP(快速数据路径)在 [Red Hat Enterprise Linux 8](https://developers.redhat.com/blog/2018/11/15/red-hat-enterprise-linux-8-beta-is-here/) 中发布，您现在可以下载并运行它。

虽然 eBPF 不是万能的，但我认为它是一个非常强大的网络调试工具，值得关注。我确信它将在未来的网络中扮演非常重要的角色。

## 问题是

我正在调试一个 [Open vSwitch](https://developers.redhat.com/search?t=Open+vSwitch) (OVS)网络问题，该问题影响了一个非常复杂的安装:一些 TCP 数据包被打乱并无序地发送，虚拟机之间的吞吐量从持续的 6 Gb/s 下降到振荡的 2–4 Gb/s。经过一些分析，结果是每个设置了 PSH 标志的连接的第一个 TCP 数据包都是无序发送的:只有第一个，而且每个连接只有一个。

我试图用两个虚拟机复制设置，在许多手册页和互联网搜索后，我发现`iptables`和`nftables`都不能破坏 TCP 标志，而`tc`可以，但它只能覆盖标志，破坏新的连接和 TCP。

或许我可以用马克、、、的组合来处理这个问题，但后来我想:这可能是 eBPF 的工作。

## eBPF 是什么？

eBPF 是 Berkeley 数据包过滤器的扩展版本。它给 BPF 增加了许多改进；最值得注意的是，它允许写入内存而不仅仅是读取内存，因此除了过滤数据包之外，它还可以编辑数据包。

eBPF 通常被称为 BPF，而 BPF 被称为 cBPF(经典 BPF)，因此根据上下文，单词 *BPF* 可以用来表示两者:在这里，我总是指扩展版本。

在幕后，eBPF 使用一个非常简单的字节码 VM，它可以执行一小部分字节码并编辑一些内存缓冲区。eBPF 有一些限制，以防止它被恶意使用:

*   循环是被禁止的，所以程序会在一个确定的时间退出。
*   它不能访问除堆栈和暂存缓冲区之外的内存。
*   只有白名单中的内核函数可以被调用。

加载的程序可以以多种方式加载到内核中，进行过多的调试和跟踪。在这种情况下，我们感兴趣的是 eBPF 如何与网络子系统一起工作。有两种方法可以使用 eBPF 程序:

*   通过 XDP 连接到物理或虚拟网卡的早期接收路径
*   通过`tc`连接到 qdisc，就像正常动作一样，在入口或出口

为了创建一个 eBPF 程序来附加，写一些 C 代码并转换成字节码就足够了。下面是一个使用 XDP 的简单例子:

```
SEC("prog")
int xdp_main(struct xdp_md *ctx)
{
    void *data_end = (void *)(uintptr_t)ctx->data_end;
    void *data = (void *)(uintptr_t)ctx->data;

    struct ethhdr *eth = data;
    struct iphdr *iph = (struct iphdr *)(eth + 1);
    struct icmphdr *icmph = (struct icmphdr *)(iph + 1);

    /* sanity check needed by the eBPF verifier */
    if (icmph + 1 > data_end)
        return XDP_PASS;

    /* matched a pong packet */
    if (eth->h_proto != ntohs(ETH_P_IP) ||
        iph->protocol != IPPROTO_ICMP ||
        icmph->type != ICMP_ECHOREPLY)
        return XDP_PASS;

    if (iph->ttl) {
        /* save the old TTL to recalculate the checksum */
        uint16_t *ttlproto = (uint16_t *)&iph->ttl;
        uint16_t old_ttlproto = *ttlproto;

        /* set the TTL to a pseudorandom number 1 < x < TTL */
        iph->ttl = bpf_get_prandom_u32() % iph->ttl + 1;

        /* recalculate the checksum; otherwise, the IP stack will drop it */
        csum_replace2(&iph->check, old_ttlproto, *ttlproto);
    }

    return XDP_PASS;
}

char _license[] SEC("license") = "GPL";
```

上面的代码片段去掉了`include`语句、助手和所有不必要的代码，是一个 XDP 程序，它将收到的 ICMP 回应回复(即 pongs)的 TTL 更改为一个随机数。main 函数接收一个`struct xdp_md`，它包含两个指向包开始和结束的指针。

要将我们的代码编译成 eBPF 字节码，需要一个支持它的编译器。Clang 支持它，并通过在编译时将`bpf`指定为目标来生成 eBPF 字节码:

```
$ clang -O2 -target bpf -c xdp_manglepong.c -o xdp_manglepong.o
```

上面的命令生成了一个看起来像是常规目标文件的文件，但是如果仔细检查，您会发现报告的机器类型将是`Linux eBPF`而不是操作系统的本机类型:

```
$ readelf -h xdp_manglepong.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Linux BPF  <--- HERE
  [...]
```

一旦被封装在一个常规的目标文件中，eBPF 程序就可以通过 XDP 加载并附加到设备上了。这可以通过使用`iproute2`套件中的`ip`来完成，使用以下语法:

```
# ip -force link set dev wlan0 xdp object xdp_manglepong.o verbose
```

这个命令指定了目标接口`wlan0`，使用`-force`选项，它将覆盖任何已经加载的现有 eBPF 代码。加载 eBPF 字节码后，这是系统行为:

```
$ ping -c10 192.168.85.1
PING 192.168.85.1 (192.168.85.1) 56(84) bytes of data.
64 bytes from 192.168.85.1: icmp_seq=1 ttl=41 time=0.929 ms
64 bytes from 192.168.85.1: icmp_seq=2 ttl=7 time=0.954 ms
64 bytes from 192.168.85.1: icmp_seq=3 ttl=17 time=0.944 ms
64 bytes from 192.168.85.1: icmp_seq=4 ttl=64 time=0.948 ms
64 bytes from 192.168.85.1: icmp_seq=5 ttl=9 time=0.803 ms
64 bytes from 192.168.85.1: icmp_seq=6 ttl=22 time=0.780 ms
64 bytes from 192.168.85.1: icmp_seq=7 ttl=32 time=0.847 ms
64 bytes from 192.168.85.1: icmp_seq=8 ttl=50 time=0.750 ms
64 bytes from 192.168.85.1: icmp_seq=9 ttl=24 time=0.744 ms
64 bytes from 192.168.85.1: icmp_seq=10 ttl=42 time=0.791 ms

--- 192.168.85.1 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 125ms
rtt min/avg/max/mdev = 0.744/0.849/0.954/0.082 ms
```

收到的每个数据包都经过 eBPF，eBPF 最终会进行一些转换，并决定丢弃或让数据包通过。

## eBPF 如何提供帮助

回到最初的网络问题，我需要破坏一些 TCP 标志，每个连接只有一个标志，`iptables`和`tc`都不允许这样做。为这个场景编写 C 代码非常简单:设置两个通过 OVS 桥连接的虚拟机，并简单地将 eBPF 连接到两个虚拟机虚拟设备中的一个。

这看起来是一个不错的解决方案，但是您必须考虑到 XDP 只支持处理收到的包，并且在接收虚拟机的`rx`路径中附加 eBPF 对交换机没有任何影响。

为了正确解决这个问题，必须使用`tc`加载 eBPF，并将其附加到 VM 中的出口路径，因为`tc`可以像任何其他操作一样将 eBPF 程序加载并附加到 qdisc。为了破坏离开主机的数据包，需要一个出口 qdisc 来连接 eBPF。

加载 eBPF 程序时，`XDP`和`tc` API 之间有一些小的区别:默认的节名不同，主函数的参数有不同的结构类型，返回值也不同，但这不是大问题。下面是一个程序的片段，当它被附加到一个`tc`动作时，会进行 TCP 篡改:

```
#define RATIO 10

SEC("action")
int bpf_main(struct __sk_buff *skb)
{
    void *data = (void *)(uintptr_t)skb->data;
    void *data_end = (void *)(uintptr_t)skb->data_end;
    struct ethhdr *eth = data;
    struct iphdr *iph = (struct iphdr *)(eth + 1);
    struct tcphdr *tcphdr = (struct tcphdr *)(iph + 1);

    /* sanity check needed by the eBPF verifier */
    if ((void *)(tcphdr + 1) > data_end)
        return TC_ACT_OK;

    /* skip non-TCP packets */
    if (eth->h_proto != __constant_htons(ETH_P_IP) || iph->protocol != IPPROTO_TCP)
        return TC_ACT_OK;

    /* incompatible flags, or PSH already set */
    if (tcphdr->syn || tcphdr->fin || tcphdr->rst || tcphdr->psh)
        return TC_ACT_OK;

    if (bpf_get_prandom_u32() % RATIO == 0)
        tcphdr->psh = 1;

    return TC_ACT_OK;
}

char _license[] SEC("license") = "GPL";
```

编译成字节码的过程和前面 XDP 的例子一样，通过以下方式完成:

```
clang -O2 -target bpf -c tcp_psh.c -o tcp_psh.o
```

但是装载不同:

```
# tc qdisc add dev eth0 clsact
# tc filter add dev eth0 egress matchall action bpf object-file tcp_psh.o
```

此时，eBPF 被加载到正确的位置，离开 VM 的包被破坏。通过检查从第二台虚拟机收到的数据包，您可以看到以下内容:

```
# tcpdump -tnni eth0 -Q in
[1579537.890082] device eth0 entered promiscuous mode
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 809667041:809681521, ack 3046223642, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 14480:43440, ack 1, length 28960
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 43440:101360, ack 1, length 57920
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [P.], seq 101360:131072, ack 1, length 29712
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 131072:145552, ack 1, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 145552:174512, ack 1, length 28960
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 174512:210712, ack 1, length 36200
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 210712:232432, ack 1, length 21720
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 232432:246912, ack 1, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [P.], seq 246912:262144, ack 1, length 15232
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 262144:276624, ack 1, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 276624:305584, ack 1, length 28960
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 305584:363504, ack 1, length 57920
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [P.], seq 363504:393216, ack 1, length 29712
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 393216:407696, ack 1, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 407696:436656, ack 1, length 28960
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 436656:494576, ack 1, length 57920
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [P.], seq 494576:524288, ack 1, length 29712
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 524288:538768, ack 1, length 14480
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 538768:567728, ack 1, length 28960
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 567728:625648, ack 1, length 57920
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [.], seq 625648:627096, ack 1, length 1448
IP 192.168.123.1.39252 > 192.168.123.2.5201: Flags [P.], seq 627096:655360, ack 1, length 28264
```

`tcpdump`确认新的 eBPF 代码正在工作，大约每 10 个 TCP 数据包中有 1 个设置了 PSH 标志。仅用 20 行 C 代码，我们就有选择地破坏了离开虚拟机的 TCP 数据包，复制了现场发生的错误，而无需重新编译任何驱动程序，甚至无需重启！这大大简化了对 [Open vSwitch fix](https://github.com/openvswitch/ovs/commit/9b4f08cdcaf253175edda088683bdd3db9e4c097) 的验证，这是其他工具无法做到的。

## 结论

eBPF 是一项相当新的技术，社区对它的采用有强烈的意见。同样值得注意的是，像 [bpfilter](https://lwn.net/Articles/747551/) 这样基于 eBPF 的项目变得越来越流行，因此，各种硬件供应商开始在他们的网卡中直接实现 eBPF 支持。

虽然 eBPF 不是灵丹妙药，不应该被滥用，但我认为它是一个非常强大的网络调试工具，值得关注。我确信它将在未来的网络中扮演非常重要的角色。

***下载*** ***[红帽企业版 Linux 8](https://developers.redhat.com/rhel8/)** **并试用 eBPF。***

## 额外资源

*   [关于 Open vSwitch 的文章](https://developers.redhat.com/search?t=Open+vSwitch)
*   [开放虚拟网络上的文章](https://developers.redhat.com/search?t=Open+Virtual+Network)
*   [介绍 stapbpf——SystemTap 的新 bpf 后端](https://developers.redhat.com/blog/2017/12/13/introducing-stapbpf-systemtaps-new-bpf-backend/)

*Last updated: January 14, 2022*
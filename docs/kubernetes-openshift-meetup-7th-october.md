# Kubernetes 和 open shift Meetup(10 月 7 日)

> 原文：<https://developers.redhat.com/blog/2017/10/20/kubernetes-openshift-meetup-7th-october>

【2017 年 10 月 7 日，本加卢鲁，一个美丽而慵懒的周六早晨。Kubernetes 来到这里，并决定这是一个伟大的一天，以满足它的一些朋友在这里，花一些时间与他们在一起。

但是唉！Kubernetes 不会说话，至少现在不会。它当然可以增长和收缩，有时注意不到，但词，不，那些失败了。它需要一些帮助，来自一些它最信任的朋友，那些它了解和理解它的朋友。

于是就有了来自[Kubernetes Bangalore](https://www.meetup.com/kubernetes-openshift-India-Meetup/events/243224993/)的朋友们，他们同意加入进来。现在，它可以为朋友们举办一场精彩的派对了。现在它有了一个举办派对的地方，由它在迈恩特拉的朋友们慷慨赞助，还有一些朋友帮助它组织。

Myntra 在新加坡有一个很酷的办公室，他们把一切都安排好了。一个漂亮的房间，配有桌子、椅子、最重要和最棒的插座、WiFi，甚至还有一个图书馆(伙计，这些家伙都是满载而归的)。最重要的是，他们甚至带来了一台摄像机，还有一个非常热情的摄影师(相信我，你稍后会看到)；)

聚会原定于上午 9:30 开始，但像往常一样，这是一个非常慵懒的星期六，而且距离相当远，朋友们一直到上午 10:30 才开始三三两两地聚会。然而，到了 10 点，因为我们有足够的法定人数，而且库伯内特斯不想在周六把人推得太远，所以派对就开放了。

派对开始时，Suraj 代表 Kubernetes 欢迎每一个到场的人，而人们则在一旁观看，有些人还在抹去他们眼中惯常的周六睡意。

首先上台的是来自 Myntra 的高级基础设施工程师 Shreehari Mohan，他解释了 Myntra 如何从 docker swarm 迁移到更棒更酷的 Kubernetes。当库伯内特听到这些时，你可以看到他脸上的喜悦。“更多的朋友”，他想，“更多的反馈，我会变得更棒”。

> 。 [@sreeharimohan](https://twitter.com/sreeharimohan?ref_src=twsrc%5Etfw) 在 [@k8sBLR](https://twitter.com/k8sBLR?ref_src=twsrc%5Etfw) meetup 上谈论 [@myntra](https://twitter.com/myntra?ref_src=twsrc%5Etfw) 从 Docker Swarm 模式到 [#kubernetes](https://twitter.com/hashtag/kubernetes?src=hash&ref_src=twsrc%5Etfw) 的转变。[pic.twitter.com/nLejsXL5m2](https://t.co/nLejsXL5m2)
> 
> —Suraj desh mukh(@ surajd _)[2017 年 10 月 7 日](https://twitter.com/surajd_/status/916521470125150208?ref_src=twsrc%5Etfw)

[//platform . Twitter . com/widgets . js](//platform.twitter.com/widgets.js)

Shreehari 首先解释了他们在开发、生产前和生产环境中面临的一些问题，以及他们如何接触 docker swarm 和最终 swarm 模式。然而，他说，群体模式网络是远远不够的。Kubernetes 在后台傻笑。“如果你去 dockins.myntra.com，你可以看到一个 UI”，Shreehari 说，与 github 上的 Dockins 项目没有任何关系，该项目由 Myntra 自己的 Dockins cli(用 go 编写)支持，底层的 [Jenkins](https://jenkins.io/) 和 [Rethinkdb](https://www.rethinkdb.com/) ， 完全抽象掉了。然而，他们仍在向 Kubernetes 迁移的过程中。

接下来是来自红帽的[维诺·亚历克斯](https://twitter.com/vinothecloudone) ，他希望人们了解[open shift Networking](https://www.openshift.org/)。“哦，有人要谈论我的哥哥了”，库伯内特兴奋地想。就这样开始了，关于 Openshift 是如何与[开放式虚拟开关](http://openvswitch.org)、、的关系 OVS 实际上是如何工作的？以及它如何作为[覆盖网络](https://en.wikipedia.org/wiki/Overlay_network) 与 OpenShift 相适应，以及它可用的各种插件。

> 。 [@vinothecloudone](https://twitter.com/vinothecloudone?ref_src=twsrc%5Etfw) 谈论 [@openshift](https://twitter.com/openshift?ref_src=twsrc%5Etfw) 在[# kubernetes](https://twitter.com/hashtag/kubernetes?src=hash&ref_src=twsrc%5Etfw)Bangalore meetup[@ k 8 sblr](https://twitter.com/k8sBLR?ref_src=twsrc%5Etfw)pic.twitter.com/59TdgGDVZ3
> 
> —Suraj desh mukh(@ surajd _)[2017 年 10 月 7 日](https://twitter.com/surajd_/status/916534841608355840?ref_src=twsrc%5Etfw)

[//platform . Twitter . com/widgets . js](//platform.twitter.com/widgets.js)

人们特别想知道推荐的 MTU 以及如何处理 IP 重复数据删除。

然而，最值得注意的问题是“OpenShift online、OpenShift.io、OpenShift dedicated 和 OCP 之间有什么区别？”感谢 [Lalatendu Mohanty](https://twitter.com/lalatenduM) ，感谢他们对此的理解和很好的解释。

简单来说， [OpenShift Origin](https://github.com/openshift/origin) 是基于 Kubernetes 的社区项目。OpenShift 容器平台作为一个 [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) 并且是一个红帽产品。Openshift Online 是 Openshift 的托管版，由红帽托管和管理；Openshift Dedicated 是一个自托管的 Openshift，由 Red Hat 工程师代表任何客户端进行管理。

[OpenshiftIO](https://openshift.io) 则是 [SaaS](https://en.wikipedia.org/wiki/Software_as_a_service) ，基本上是一个在线托管(在 OpenShift 上)开发平台，用于规划、创建和部署托管云服务。

这就把我们带到了派对前半部分的尾声，每个人都起来休息了。“该起床了，和一些朋友单独在一起”，库贝内特斯想，事实也的确如此，一边喝着可乐，一边吃着薯片。

接下来是来自 Arvind Internet 的 [Chandresh Pancholi](https://www.linkedin.com/in/chandresh-pancholi-467a8015/) ， ，他想谈谈他们从虚拟机到容器的旅程。“对我来说，这是一个有趣的例子，人们实际上在移动他们基于虚拟机的东西。这不正是大多数人所期望的我的用户吗？他解释说，仅仅是搬到 kubernetes，他们就从炮轰拉赫到数万人。

> 从 VM 到容器的旅程( [#Kubernetes](https://twitter.com/hashtag/Kubernetes?src=hash&ref_src=twsrc%5Etfw) )作者 Chandresh[@ k8sBLR](https://twitter.com/k8sBLR?ref_src=twsrc%5Etfw)[@ my ntra _ engg](https://twitter.com/myntra_engg?ref_src=twsrc%5Etfw)[pic.twitter.com/reYs5N1E16](https://t.co/reYs5N1E16)
> 
> — Suraj Narwade？？(@ red _ suraj)[2017 年 10 月 7 日](https://twitter.com/red_suraj/status/916550935177863168?ref_src=twsrc%5Etfw)

[//platform . Twitter . com/widgets . js](//platform.twitter.com/widgets.js)

他们有 50 多项服务，由一个数据库支持，并通过 [ELB](https://aws.amazon.com/documentation/elastic-load-balancing/) 公开。他们试图转移到[弹性豆茎](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html) ，合并服务甚至[云代工厂](https://www.cloudfoundry.org/foundation/) 但收效甚微。Kubernetes 微笑着，想起了这个故事。这是在它更年轻的时候，准确地说是 1.3 版本，他们找到了它，它发挥了它的魔力。它将他们的需求从 200 个节点减少到大约 20-30 个。它和它的一个密友，[的掌舵人](https://helm.sh/) 曾经做过这件事。他们让这家初创公司不仅节省了资金，还轻松管理和部署了他们的设置。

在此之后，来自 Formcept 的 Sahil Sharma ， 来了，他希望人们像他一样了解 Kubernetes。就像真正了解它一样。而他正是这么做的， [Kubernetes，敬酒不吃吃罚酒](https://github.com/kelseyhightower/kubernetes-the-hard-way) 。这是库伯内特斯的内心深处的东西，因为它转过身来，隐藏着喜悦的泪水，内心微笑着，因为它看到了爱。

> 。 [@gh05t_r00t](https://twitter.com/gh05t_r00t?ref_src=twsrc%5Etfw) 谈安装 [#Kubernetes](https://twitter.com/hashtag/Kubernetes?src=hash&ref_src=twsrc%5Etfw) 来硬的！在 [@k8sBLR](https://twitter.com/k8sBLR?ref_src=twsrc%5Etfw) meetup。[pic.twitter.com/NAyW61H7hU](https://t.co/NAyW61H7hU)
> 
> —Suraj desh mukh(@ surajd _)[2017 年 10 月 7 日](https://twitter.com/surajd_/status/916560192120143872?ref_src=twsrc%5Etfw)

[//platform . Twitter . com/widgets . js](//platform.twitter.com/widgets.js)

派对就这样结束了。一些 OpenEBS 的朋友来了，给帮助 Kubernetes 举办如此棒的派对的人分发了 [HacktoberFest](https://hacktoberfest.digitalocean.com/) 好东西。

人群令人敬畏，人群中既有初学者也有经验丰富的人。遇到了一些很棒的人。建立并改革了伟大的联系。

当然，在那之后有一个秘密的会后派对，Kubernetes 和它在这里的一些最亲密的朋友去了一个很棒的午餐去了[Punjabi Rasoi](https://www.google.co.in/maps/place/The+Punjabi+Rasoi/@12.9118269,77.6362494,17z/data=!3m1!4b1!4m5!3m4!1s0x3bae149055555549:0x9bcf2312a7a0673f!8m2!3d12.9118269!4d77.6384381?dcr=0)T3。

在班加罗尔凉爽的微风和细雨下，库伯内特心里想:“我真幸运，有这么好的朋友。我爱这些家伙”，当这个令人敬畏的周六拉开帷幕时，人们已经开始期待他的下一场派对了...

我在这里附上了 meetup 的视频链接以供观看， [Kubernetes 和 OpenShift Meetup](https://www.youtube.com/playlist?list=PLoRxzlVGlD2jzWvYB3nXoSAOytmgTNcDC) 。

* * *

**[**红帽 OpenShift 容器平台**](https://www.openshift.com/container-platform/) **可供下载。****

***Last updated: September 3, 2019***
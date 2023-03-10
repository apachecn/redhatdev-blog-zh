# 容器运行时的安全性考虑

> 原文：<https://developers.redhat.com/blog/2018/12/19/security-considerations-for-container-runtimes>

https://www.youtube.com/watch?v=HIM0HwWLJ7g

我演讲的录音*容器运行时的安全考虑* - [丹·沃什](https://developers.redhat.com/blog/author/rhatdan/)，红帽( [@rhatdan](https://twitter.com/rhatdan) )

解释/演示在您的容器环境中使用具有不同安全特性的 Kubernetes

一般概念

*   运行没有根的容器，句号
*   利用主机提供的所有安全功能

配置 CRI-O:

*   运行包含只读图像的容器
*   限制容器中运行的 Linux 功能
*   设置容器存储，以更安全的方式修改存储选项
*   配置可选的 OCI 运行时:Kata、Gvisord 和 Nabla 来运行锁定的容器

构建考虑安全性的图像。

*   限制包装/容器图像的攻击面
*   在锁定的 kubernetes 容器中构建容器映像

用户名称空间的进步

*   演示使用不同的用户名称空间运行每个容器
*   配置系统，以利用用户名称空间容器分离，而不会大幅降低速度

还有更多...

您可能会发现 [Scott McCarty 的](https://developers.redhat.com/blog/author/fatherlinux/)文章 [*对容器术语*](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/) 的实用介绍有助于比较容器运行时。

另见 [*没有守护进程的容器:* *红帽企业版 Linux 7.6 和红帽企业版 Linux 8 Beta*](https://developers.redhat.com/blog/2018/11/20/buildah-podman-containers-without-daemons/) 中可用的 Podman 和 Buildah。

*Last updated: September 3, 2019*
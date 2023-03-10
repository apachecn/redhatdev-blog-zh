# Kubernetes 会拖慢我的数据库吗？

> 原文：<https://developers.redhat.com/blog/2018/12/18/will-kubernetes-slow-down-my-database>

https://www.youtube.com/watch?v=k-jAzGzhMXk

来自 KubeCon 2018 的闪电谈话:我的数据库有多快？- [乔希·伯克思](https://twitter.com/fuzzychef)，红帽

我知道我的数据库在 Kubernetes 和云本地存储上会慢一些，但是会慢多少呢？这是每个考虑将数据库等传统托管的有状态服务迁移到 Kubernetes 的人一直在问的问题。直到现在，我们还没有很好的答案。本演示将详细介绍一系列在 Kubernetes 上运行的 PostgreSQL 微基准测试，这些微基准测试采用各种配置，包括裸机、本地存储、gluster 和 rook。您将对抽象出您的存储问题的延迟和吞吐量成本有一个明确的概念，并能够自己做出平台决策。

*Last updated: August 26, 2019*
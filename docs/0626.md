# 使用 Red Hat OpenShift 优化运算符进行弹性搜索

> 原文：<https://developers.redhat.com/blog/2019/11/12/using-the-red-hat-openshift-tuned-operator-for-elasticsearch>

最近，我帮助一个客户在 Kubernetes (ECK)的 Red Hat open shift 4 . x(T1)上部署了弹性云。他们遇到了一个问题，Elasticsearch 会抛出一个类似如下的错误:

```
Max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

根据官方文档， [Elasticsearch](https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html) 默认使用一个`mmapfs`目录来存储其索引。mmap 计数的默认操作系统限制可能太低，这可能导致内存不足异常。通常，管理员会通过运行以下命令来增加限制:

```
sysctl -w vm.max_map_count=262144
```

然而，OpenShift 使用 [Red Hat CoreOS](https://www.openshift.com/learn/coreos/) 作为其工作节点，因为它是一个自动更新的、运行容器化工作负载的最小操作系统，所以您不应该手动登录工作节点并进行更改。这种方法是不可扩展的，会导致工作节点被感染。相反，OpenShift 通过其[节点调优操作符](https://docs.openshift.com/container-platform/4.2/nodes/nodes/nodes-node-tuning-operator.html)提供了一种优雅且可伸缩的方法来实现同样的功能。

默认的优化配置包含一个用于 Elasticsearch 的配置文件。给定节点上的 tuned 操作符查找在相同节点上运行的带有 tuned.openshift.io/elasticsearch 标签集(匹配)的 pod。如果找到，它应用`sysctl`命令(数据)。

您可以通过登录到 OpenShift 集群并运行以下命令来查看默认配置:

```
bastion $ oc get Tuned/default -o yaml -n openshift-cluster-node-tuning-operator

apiVersion: tuned.openshift.io/v1alpha1
kind: Tuned
metadata:
  name: default
  namespace: openshift-cluster-node-tuning-operator
spec:
  profile:

...
...

  - name: "openshift-node-es"
    data: |
      [main]
      summary=Optimize systems running ES on OpenShift nodes
      include=openshift-node
      [sysctl]
      vm.max_map_count=262144
  recommend:

...
...

  - profile: "openshift-node-es"
    priority: 20
    match:
    - label: "tuned.openshift.io/elasticsearch"
      type: "pod"

```

```

诀窍是确保 Elasticsearch 操作者用标签标记它的 pod:`tuned.openshift.io/elasticsearch`。下面是如何实现这一点的示例。

```
---
apiVersion: elasticsearch.k8s.elastic.co/v1alpha1
kind: Elasticsearch
metadata:
  name: elasticsearch-tst
spec:
  version: "7.2.0"
  setVmMaxMapCount: false
  nodes:
  - config:
      node.master: true
      node.data: true
    nodeCount: 1
    podTemplate:
      metadata:
        labels:
          tuned.openshift.io/elasticsearch: ""

```

调优的操作员将读取 pod 标签`tuned.openshift.io/elasticsearch`并将`vm.max_map_count=262144`添加到运行 pod 的节点。这很有用，因为可以在集群的不同节点上终止和调度 pod。不再需要手动担心运行特定工作负载的节点的 sysctl 配置。

感谢詹姆斯·赖尔斯帮助解决这个问题。

如果你遇到任何问题，请告诉我。

*Last updated: July 1, 2020*
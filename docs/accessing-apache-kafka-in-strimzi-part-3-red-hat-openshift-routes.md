# 在 Strimzi 中访问 Apache Kafka:第 3 部分 Red Hat OpenShift 路线

> 原文：<https://developers.redhat.com/blog/2019/06/10/accessing-apache-kafka-in-strimzi-part-3-red-hat-openshift-routes>

在这个系列文章的第三部分(见下面以前文章的链接)，我们将看看 [Strimzi](https://strimzi.io/) 如何使用 [Red Hat OpenShift routes](https://developers.redhat.com/openshift/) 暴露[阿帕奇卡夫卡](https://developers.redhat.com/videos/youtube/CZhOJ_ysIiI/)。本文将解释路线是如何工作的，以及如何与 Apache Kafka 一起使用。路线只在 OpenShift 上可用，但如果你是 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 用户，也不要难过；本系列的下一篇文章将讨论使用 Kubernetes Ingress，它类似于 OpenShift 路由。

**注:**strim zi 和 Apache Kafka 项目的产品化和支持版本作为[红帽 AMQ](https://www.redhat.com/en/technologies/jboss-middleware/amq) 产品的一部分提供。

## Red Hat OpenShift 路线

路由是一个 OpenShift 概念，用于向 Red Hat OpenShift 平台外部公开服务。路由处理数据路由和 DNS 解析。DNS 解析通常使用[通配符 DNS 条目](https://en.wikipedia.org/wiki/Wildcard_DNS_record)来处理，这允许 OpenShift 根据通配符条目为每个路由分配自己的 DNS 名称。用户不必做任何特殊的事情来处理 DNS 记录。如果您没有任何可以设置通配符条目的域，OpenShift 可以使用诸如 [nip.io](https://nip.io/) 之类的服务进行通配符 DNS 路由。数据路由使用 [HAProxy](https://www.haproxy.org/) 负载均衡器完成，它充当域名背后的路由器。

路由器的主要用例是 HTTP(S)路由。这些路由能够对 HTTP 和 HTTPS(带 TLS 终端)流量进行基于路径的路由。在这种模式下，HTTP 请求将根据请求路径被路由到不同的服务。但是，因为 Apache Kafka 协议不是基于 HTTP 的，所以 HTTP 特性对于 Strimzi 和 Kafka 代理来说不是很有用。

幸运的是，这些路由也可以用于 TLS 传递。在这种模式下，它使用 TLS 服务器名称指示( [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) )来确定流量应该路由到的服务，并将 TLS 连接传递到该服务(并最终传递到支持该服务的 pod)，而不对其进行解码。这种模式正是斯特里姆齐用来揭露卡夫卡的。

如果你想了解更多关于 OpenShift 路线的信息，请查阅 [OpenShift 文档](https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html)。

## 使用 OpenShift 路线曝光卡夫卡

使用 OpenShift routes 公开 Kafka 可能是所有可用侦听器类型中最简单的。您需要做的就是在 Kafka 自定义资源中配置它。

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    # ...
    listeners:
      # ...
      external:
        type: route
    # ...

```

Strimzi Kafka 操作员和 OpenShift 将处理剩下的工作。为了提供对单个代理的访问，我们使用了与节点端口相同的技巧，这些技巧在[上一篇文章](https://developers.redhat.com/blog/?p=601137)中有所描述。我们为每个经纪人创建一个专门的服务，它将用于直接处理单个经纪人。除此之外，我们还将使用一个服务来引导客户端。该服务将在所有可用的 Kafka 经纪人之间循环。

与使用节点端口不同，这些服务将只是常规的`clusterIP`类型的 Kubernetes 服务。Strimzi Kafka 运营商还将为这些服务中的每一个创建一个`Route`资源，这将使用 HAProxy 路由器公开它们。Strimzi 将使用分配给这些路由的 DNS 地址来配置不同 Kafka 代理中的广告地址。

![Strimzi Route access](img/6dcc9f1f7ac49121ed6df5b83f592897.png)

Kafka 客户端将连接到 bootstrap 路由，该路由将通过 bootstrap 服务将它们路由到其中一个代理。从这个代理中，客户端将获得元数据，该元数据将包含每个代理路由的 DNS 名称。Kafka 客户端将使用这些地址连接到专用于特定代理的路由，路由器将再次通过相应的服务将其路由到正确的 pod。

如前一节所述，路由器的主要用例是路由 HTTP(S)流量。因此，它总是监听端口 80 和 443。因为 Strimzi 正在使用 TLS 直通功能，所以以下情况将为真:

*   端口将始终为 443，作为用于 HTTPS 的端口。
*   流量将**始终使用 TLS 加密**。

获取与您的客户端连接的地址很容易。如前所述，端口将始终是 443。当用户尝试连接到端口 9094 而不是 443 时，这可能会导致问题。但是，对于 OpenShift 路由，443 始终是正确的端口号。您可以在`Route`资源的状态中找到主机(用您的集群的名称替换`my-cluster`):

```
oc get routes my-cluster-kafka-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'

```

默认情况下，路由的 DNS 名称将基于它所指向的服务的名称和 OpenShift 项目的名称。例如，对于我在名为`myproject`的项目中运行的名为`my-cluster`的 Kafka 集群，默认的 DNS 名称将是`my-cluster-kafka-bootstrap-myproject.<router-domain>`。

因为流量总是使用 TLS，所以您必须总是在 Kafka 客户端中配置 TLS。这包括从代理获取 TLS 证书，并在客户端对其进行配置。您可以使用以下命令获取 Kafka 代理使用的 CA 证书，并将其导入 Java 密钥库文件，该文件可用于 Java 应用程序(用您的集群的名称替换`my-cluster`):

```
oc extract secret/my-cluster-cluster-ca-cert --keys=ca.crt --to=- > ca.crt
keytool -import -trustcacerts -alias root -file ca.crt -keystore truststore.jks -storepass password -noprompt

```

有了证书和地址，您就可以连接到 Kafka 集群。以下示例使用了属于 Apache Kafka 的`kafka-console-producer.sh`实用程序:

```
bin/kafka-console-producer.sh --broker-list :443 --producer-property security.protocol=SSL --producer-property ssl.truststore.password=password --producer-property ssl.truststore.location=./truststore.jks --topic 

```

有关更多详情，请参见 [Strimzi 文档](https://strimzi.io/docs/latest/full.html#proc-accessing-kafka-using-routes-deployment-configuration-kafka)。

## 自定义

如前一节所述，默认情况下，路由会根据您的集群名称和命名空间自动分配 DNS 名称。但是，您可以对此进行自定义，并指定您自己的 DNS 名称:

```
# ...
listeners:
  external:
    type: route
    authentication:
      type: tls
    overrides:
      bootstrap:
        host: bootstrap.myrouter.com
      brokers:
      - broker: 0
        host: broker-0.myrouter.com
      - broker: 1
        host: broker-1.myrouter.com
      - broker: 2
        host: broker-2.myrouter.com
# ...

```

定制的名称仍然需要匹配 OpenShift 路由器的 DNS 配置，但是您可以给它们一个更友好的名称。当然，自定义 DNS 名称(以及自动分配给路由的名称)将被添加到 TLS 证书中，您的 Kafka 客户端可以使用 TLS 主机名验证。

## 利弊

路线仅在 Red Hat OpenShift 上可用。所以，如果你正在使用 Kubernetes，这显然是一个破坏交易的劣势。另一个潜在的缺点是路由总是使用 TLS 加密。在 Kafka 客户端和应用程序中，您将始终需要处理 TLS 证书和加密。

您还需要仔细考虑性能。OpenShift HAProxy 路由器将作为您的 Kafka 客户和经纪人之间的中间人。这种方法会增加延迟，还会成为性能瓶颈。使用 Kafka 的应用程序通常会产生大量流量——每秒数百甚至数千兆字节。请记住这一点，并确保 Kafka 流量仍将为使用路由器的其他应用程序留下一些容量。幸运的是，OpenShift 路由器具有可伸缩性和高度可配置性，因此您可以微调其性能，如果需要，甚至可以为 Kafka 路由设置一个单独的路由器实例。

使用 Red Hat OpenShift 路线的主要优点是它们很容易工作。与上一篇文章中讨论的节点端口不同，这些节点端口通常很难配置，并且需要对 Kubernetes 和基础设施有更深入的了解，OpenShift 路由在任何 OpenShift 安装上开箱即可非常可靠地工作。

### 阅读更多

*   [在 Strimzi 中访问阿帕奇卡夫卡:第 1 部分-简介](https://developers.redhat.com/blog/?p=601077)
*   [访问 Strimzi 中的 Apache Kafka:第 2 部分-节点端口](https://developers.redhat.com/blog/?p=601137)
*   [在 Strimzi 中访问阿帕奇卡夫卡:第 3 部分-红帽 OpenShift 路线](https://developers.redhat.com/blog/?p=601277)
*   [在 Strimzi 中访问 Apache Kafka:第 4 部分-负载平衡器](https://developers.redhat.com/blog/?p=601357)
*   [在《斯特里姆齐:第五部分——入口》中访问阿帕奇卡夫卡](https://developers.redhat.com/blog/?p=601457)

*Last updated: March 18, 2020*
# 在 Red Hat OpenShift 上部署 Mosquitto MQTT 消息代理，第 2 部分

> 原文：<https://developers.redhat.com/blog/2021/04/26/deploying-the-mosquitto-mqtt-message-broker-on-red-hat-openshift-part-2>

本文的前半部分介绍了[mosquito](https://mosquitto.org/)[消息队列遥测传输](https://mqtt.org/) (MQTT)消息代理，并展示了如何将 mosquito 构建成适合在[容器](/topics/containers)中使用的映像。在本文的后半部分，您将配置 Mosquitto 映像并将其部署到运行在 [Red Hat OpenShift](/products/openshift/overview) 上的应用程序中。您可以从我的 [GitHub 存储库](https://github.com/kevinboone/mosquitto-openshift)中获得示例文件。

## 配置 OpenShift

假设您已经阅读了第 1 部分，那么您应该在存储库中有了基本的 Mosquitto 映像，在 OpenShift 上部署时可以参考它的 URI。如果您熟悉 Docker 和 Podman 工具系列，以及您想要部署的软件，那么配置和部署并不是特别困难。我花在实现、构建和测试图像上的时间比在本文中描述如何做要少。

我认为将映像部署到 OpenShift 最广泛兼容的方式是使用 YAML 格式的部署配置。当然，还有其他部署映像的方法，但是本文中的步骤不需要在任何 OpenShift 版本上进行修改就可以工作。

我将描述两个部署。第一种方法仅使用图像中的默认值。这应该提供一些你可以测试的东西。然后，我将描述如何覆盖配置文件来创建特定于站点的安装。

## 在 OpenShift 上部署默认映像

首先，您需要创建一个 YAML 部署配置。请注意，下面的代码片段不是完整的部署配置:我已经删除了大部分样板代码，只提取了特定于该应用程序的内容。完整的 YAML 文件在源代码库中。该文件指定了两个服务以及部署配置:一个服务用于明文端口 1883，另一个用于 TLS 端口 8883。公开的端口也必须在部署配置中列出，但是这本身并不能创建其他应用程序可以连接到的服务—您还需要服务:

```
kind: DeploymentConfig
apiVersion: apps.openshift.io/v1
metadata:
  name: mosquitto-ephemeral
spec:
  replicas: 1
    spec:
      containers:
          name: mosquitto-ephemeral
          image: quay.io/kboone/mosquitto-ephemeral:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 1883
              protocol: TCP
            - containerPort: 8883
              protocol: TCP
          resources:
            limits:
              memory: 128Mi
---
kind: Service
metadata:
  name: mosquitto-ephemeral-tcp
spec:
  ports:
      port: 1883
      targetPort: 1883
---
kind: Service
metadata:
  name: mosquitto-ephemeral-tls
spec:
  ports:
      port: 8883
      targetPort: 8883

```

部署配置指定了要下载的映像(使用其存储库 URI)、副本的数量(在本例中必须是 1 才有用)和内存限制。Mosquitto 通常不需要太多内存，所以我设置了 128 MB 的限制。实际上，使用接近这么大内存的任何东西都需要大量的客户端负载。完整的 YAML 部署配置指定了活跃度和就绪性探测以及其他配置，这些配置通常很重要，但与本文无关。

要部署 YAML 文件，从而创建 pod 和服务，请输入:

```
$ oc apply -f mosquitto-ephemeral.yaml

```

过一会儿——实际上*应该是一会儿，因为从存储库中取出的映像只有 7mb——您应该会看到 pod 正在运行:*

```
$ oc get pods
NAME READY STATUS RESTARTS AGE
mosquitto-ephemeral-1-5clnd 1/1 Running 1 23h

```

您可以检查以确保存在针对明文和传输层安全(TLS)端口的服务，如下所示:

```
$ oc get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
mosquitto-ephemeral-tcp ClusterIP 172.30.207.29 <none> 1883/TCP 24h
mosquitto-ephemeral-tls ClusterIP 172.30.163.226 <none> 8883/TCP 24h

```

此时，集群中的其他 pod 应该能够使用服务主机名`mosquitto-ephemeral-tcp`和`mosquitto-ephemeral-tls`通过适当的端口连接到 MQTT 代理。

### 定义路线

为了让 MQTT 客户机能够从 OpenShift 集群外部连接到 broker pod，您必须定义一个带有“直通”终止模式的 TLS 加密路由。OpenShift 路由器将不能路由明文 MQTT 连接，因为 MQTT 协议不像 HTTP 那样在其“Host:”报头中携带目的地主机名。相反，路由器将通过检查 TLS 握手中的服务器名称标识(SNI)信息来识别目标 pod。在直通模式下，路由器将 TLS 握手传递给目标 pod，因此客户端收到的证书将是 pod 的证书，而不是 OpenShift 路由器的证书。客户端必须信任的是安装在部署的映像中的 pod 证书。

因此，要创建路由，请输入:

```
$ oc create route passthrough --service=mosquitto-ephemeral-tls \
        --port 8883 --hostname=mosquitto.apps.my_domain
```

请注意，所选的主机名必须由您的 DNS 配置映射到 OpenShift 路由器，并且客户端必须使用该确切名称连接到 OpenShift 路由器。请记住，正是这个出现在 TLS 握手中的主机名允许路由器找到正确的 pod。

### 从映像中获取 CA 证书

要从 OpenShift 集群外部连接 MQTT 客户机，您需要来自映像的 CA 证书。如果没有创建证书的源代码，如何从映像中获取证书呢？这个问题在实践中不应该出现，因为部署人员已经用特定于站点的证书替换了映像中的默认证书。但是，如果您必须这样做，您可以直接从 pod 获取 CA 证书，如下所示。

首先，获取客户端想要连接的 pod 的编号:

```
$ oc get pod

```

然后，发出以下命令，将斜体的 *podnum* 更改为您刚刚获得的 pod 的编号:

```
$ oc cp mosquitto-ephemeral-1-*podnum*:/myuser/ca.crt mosquitto_ca.crt

```

### 测试安装

检索到证书(或从源代码存储库中复制)后，您可以使用其主机名和端口 443(这是 OpenShift 路由器的默认 TLS 端口，而不是 pod 的 TLS 端口，客户端将永远看不到它)在 pod 中测试 Mosquitto 代理:

```
$ mosquitto_pub -t foo -m "text" --cafile mosquitto_ca.crt \
  --insecure -u admin -P admin --host mosquitto.apps.my_domain --port 443

```

在这个命令中仍然需要`--insecure`，因为证书中的主机名(`acme.com`)与客户端提供的主机名不匹配。事实上，您总是需要禁用主机名验证，除非您生成一个服务器证书，其主机名与您自己的安装中的`mosquitto.apps.my_domain`或等同物相匹配。

## 在 OpenShift 上部署自定义映像

前面的步骤——同样，写起来比做起来花的时间长——使用默认设置部署了 Mosquitto 图像；也就是说，设置被烘焙到图像中。在实践中，一些特定于站点的配置几乎总是必要的。在这一节中，我将向您展示如何提供一个定制的凭证文件来指定一个非默认用户。假设您可以访问部署配置，那么您可以使用相同的机制来覆盖映像中的任何文件。我选择了凭证文件，因为用它来演示原理只需要替换一个文件；要更改 TLS 证书，我们必须替换所有三个证书。

### 为什么不直接编辑 pod 中的配置文件呢？

不熟悉 OpenShift 的开发人员有时会问，为什么他们不能登录到一个 pod 并编辑配置文件。这有两个原因:一个是实际的，一个是哲学的。

实际原因是 pod 的默认用户(`myuser`)没有相关文件的写权限。它们归`root`所有，而且是故意的；这不是管理安装的好方法。可以(尽管强烈建议)更改这些配置文件的权限。然后，管理员可以登录到 pod，编辑文件，并向代理发送 SIGHUP 信号，使其重新加载配置。

更哲学的原因是，管理员与 OpenShift pods 的关系应该是*对牛，而不是对宠物*。这个比喻指的是豆荚的可替代性。OpenShift 基础架构可以随时终止和重启 pod。发生这种情况时，pod 的文件系统将从映像中恢复，任何手动更改都将丢失。

因此，对 pod 进行配置更改的安全方法是更改构建 pod 的规范。在这里，这意味着更改部署配置。使用这种方法，从该部署配置构建的所有吊舱将是相同的。

### 指定一个非默认用户

从概念上讲，更改映像中的身份验证配置最简单的方法就是覆盖现有的`/myuser/passwd`文件。对于管理员来说，这不一定是最简单的方法，但可能是最容易理解的。为此，我们将把新文件插入到 OpenShift configmap 中，然后修改部署配置，以便新文件在启动时插入到 pod 的文件系统中。

首先，使用新的用户和密码创建一个新的凭据文件:

```
$ touch passwd
$ mosquitto_passwd -b passwd foo foo

```

这个凭证文件定义了一个用户`foo`，密码为`foo`。

现在，从文件`passwd`创建一个名为`passwd`的配置映射:

```
$ oc create configmap passwd --from-file=passwd=./passwd

```

配置映射和文件不必具有相同的名称。事实上，如果需要，您可以在同一个 configmap 中存储多个文件。

### 修改部署配置

现在，您需要修改部署配置，以便它从 configmap 插入`passwd`文件，替换映像中的默认文件。和以前一样，我只展示 YAML 文件的相关部分；完整的文件在源存储库中是`mosquitto-ephemeral-passwd.yaml`:

```
spec:
     containers:
        volumeMounts:
          - name: passwd-mount
            mountPath: /myuser/passwd
            subPath: passwd

    volumes:
        configMap:
          name: passwd-mount
          items:
            - key: passwd
              path: passed

```

`volumeMounts`部分定义了将要挂载的文件的路径(即，使其可用)，并标识了这些文件的源。应该有一个相应的`volumes`部分来定义实际的源。在这种情况下，文件的源是一个配置映射。名称`passwd-mount`将待挂载的文件与该文件的源相关联。

您可以使用相同的技术来提供映像中任何其他文件的定制版本，包括 TLS 证书。

### 文件替换的替代方法

这种文件挂载技术的一个优点是，如果部署配置中没有映射，映像将返回到其默认值。因此，在没有复杂配置的情况下，有可能得到一些用于测试目的的东西。

在实践中，对于管理员来说，这种文件替换方法可能会很笨拙且容易出错，尤其是在不完全清楚需要替换哪些文件以及它们的格式是什么的时候。映像的维护者需要提供非常清晰的文档，说明部署者可能需要提供的每个文件的用途、格式和位置。

另一种方法是将特定的配置属性(而不是整个文件)放入配置图中，并让 pod 的启动脚本在运行时解包配置图并构建一组配置文件。

事实上，可以完全省略 configmap，直接在部署配置中指定环境属性。这些属性被注入到启动脚本可以读取的普通 Linux 环境变量中。这个脚本类似于前面描述的技术，可以在运行时从属性构建配置文件。

一种更加优雅的方法是使用[应用程序模板](https://docs.openshift.com/container-platform/4.5/openshift_images/using-templates.html)。模板提供了一种基于可以在 OpenShift 控制台的 web 表单中输入的值来生成部署配置的方法。我不会详细描述这种方法，因为大多数 Red Hat 的 OpenShift 产品正在从使用模板转向使用*操作符。*运营商提供了一种非常强大的方式来配置复杂的软件安装，但 Mosquitto 消息代理的安装似乎不太可能需要这种复杂性，或者从中受益。

## 第二部分的结论

本文向您介绍了 Mosquitto MQTT 消息代理，它广泛用于[物联网](https://developers.redhat.com/blog/category/iot/) (IoT)和遥测应用。我使用 Mosquitto 演示了如何打包和部署不是为打包而设计的包。正如您所看到的，OpenShift 可以与各种各样的工具配合使用，用于加载图像、认证用户和其他任务。

*Last updated: October 14, 2022*
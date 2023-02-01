# 在 Red Hat OpenShift 上部署 Mosquitto MQTT 消息代理，第 1 部分

> 原文：<https://developers.redhat.com/blog/2021/04/16/deploying-the-mosquitto-mqtt-message-broker-on-red-hat-openshift-part-1>

Mosquitto 是一个轻量级的消息代理，它支持 T2 消息队列遥测传输协议。Mosquitto 广泛应用于[物联网](/blog/category/iot/) (IoT)和遥测应用中，在这些应用中，像[红帽 AMQ](/products/amq/overview) 这样的全功能消息代理会是不必要的负担。Mosquitto 还在分布式系统中充当进程间通信的消息总线。由于避免了复杂的功能，Mosquitto 易于调整，并且可以用相对适中的内存和 CPU 资源处理大量的应用程序工作负载。

让 Mosquitto 在 [Red Hat OpenShift](/products/openshift/overview) 上可用主要有两个阶段。首先，你需要以一种与 OpenShift 广泛兼容的方式[容器化](/topics/containers)应用程序。容器化的一部分包括在存储库中安装容器映像，OpenShift 可以从存储库中下载它。其次，您需要在 pod 中部署容器化的映像，提供特定安装所需的任何属性和配置。本文的前半部分展示了如何将 Mosquitto 构建成适合在容器中使用的图像。后半部分将向您展示如何在 OpenShift 上配置和部署 Mosquitto 映像。

## 蚊子基础知识

Mosquitto 的消息模型是发布-订阅，就像 Java 消息服务(JMS)术语中的“主题”。事实上，Mosquitto 文档使用术语“主题”来表示其消息目的地。在 AMQ 7(以及 AMQ 7 所基于的 ActiveMQ Artemis)的术语中，Mosquitto 目的地默认为*非持久任播地址*。Mosquitto 只通过 MQTT 进行通信:这是一种轻量级、负载中立、非事务性的有线协议。

**注意**:mosquito 的详细描述超出了本文的范围。更多信息请见 [Mosquitto 网站](https://mosquitto.org/)。

在 OpenShift pod 中部署 Mosquitto 相对容易:Mosquitto 没有(也不需要)集群支持，因此它适合简单的部署方法。虽然您可能会在 OpenShift 上找到 Mosquitto 的用途，但本文重点关注的是与 OpenShift 兼容的软件打包，就像关注 Mosquitto 本身一样。许多广泛使用的软件包不是为云或容器操作而设计的；因此，它们给创建可维护、可配置的安装带来了挑战。

## 使用带 openshift 的蚊子

在本文中，我对如何在 OpenShift 上使用 Mosquitto(或任何其他轻量级应用程序)做了一些假设:

*   您希望最小化 pod 启动时间和容器图像大小。运行 Mosquitto 的多个副本没有任何好处，因此在发生崩溃时，您需要一个新的 pod 在几毫秒内启动。
*   该应用程序既可以在 OpenShift 集群内部使用服务，也可以在外部使用路由。与 OpenShift 路由框架的兼容性要求在有线协议中使用传输层安全性(TLS)加密，我将在后面讨论。
*   对于集群间客户端和外部客户端，客户端访问将进行身份验证，而不是完全开放。在这个例子中，我将使用简单的用户密码认证，尽管 Mosquitto 也支持客户端证书。
*   尽管安装程序需要在 OpenShift 上定制部署，但是只使用容器映像中的缺省值的安装应该足以进行测试。不应该仅仅为了开始而强制提供复杂的配置。例如，这意味着为容器映像提供默认的 TLS 证书。

在说明中，我还假设您在本地工作站上安装了 Mosquitto——或者至少安装了 Mosquitto 测试客户端——用于测试。在 Fedora 和[Red Hat Enterprise Linux](/products/rhel/overview)(RHEL)系统上，在 shell 中键入`dnf install mosquitto`应该就可以了。

为了简洁起见，我不会完整地显示所有相关文件。你可以从我的 [GitHub 库](https://github.com/kevinboone/mosquitto-openshift)获得所有这些文件。

注意，在我的演示中，我将使用 [Podman](https://podman.io/) 来构建和运行图像。Docker 和 Buildah 也应该可以工作，而无需修改示例。

## 集装箱化的配置文件

Mosquitto 至少需要四个文件或文件集，才能在 OpenShift 上提供基本的服务。您可能需要在安装时配置这些文件:

*   至少一个用户的凭据
*   一套 TLS 证书
*   代理的配置文件
*   启动脚本

标准的 Mosquitto 安装为配置文件提供了默认位置。然而，在本例中，我选择将所有可能需要部署人员修改的文件放在容器映像的一个目录中。我已经为将拥有代理的用户帐户选择了主目录。我正在使用目录`/myuser`。该目录的名称是任意的，但是如果安装程序需要更改的所有文件都在同一个目录中，则管理会更简单。

在源存储库中，所有相关的配置文件和证书都在`files`目录中，并且在构建时会被复制到映像中的`/myuser`中。

我们还需要一个 Dockerfile 来从基础映像、Mosquitto 二进制文件和配置文件构建容器映像。在将生成的图像安装到 OpenShift 上之前，可以在本地工作站上测试它(也许*必需的*是一个更好的词)。

### 凭证文件

Mosquitto 提供了一个简单的基于凭证文件的用户密码认证机制。凭证文件采用专有格式，包含哈希密码。Mosquitto 提供了一个名为`mosquitto_passwd`的实用程序来编辑文件。

默认图像在名为`passwd`的凭证文件中为一个名为`admin`的用户提供密码`admin`。我创建了这个文件，如下所示:

```
$ touch files/passwd
$ mosquitto_passwd -b files/passwd admin admin

```

**注意**:在第 2 部分中，我将解释在 OpenShift 上部署映像时如何提供不同的凭证。

要创建具有不同权限的多个用户，请编辑主配置文件并将用户添加到凭据文件中。

### TLS 证书

Mosquitto 要求实际安装至少有三个证书。像安装程序可能覆盖的所有其他文件一样，这些文件将被安装到`/myuser`目录中的映像中。这些证书是:

*   一个根证书颁发机构(CA)证书，所有其他证书将根据该证书进行身份验证。在我的例子中，这个文件被命名为`ca.crt`。*您必须与客户*共享此证书。在示例中，我使用 OpenSSL 生成这个文件，但是在生产安装中，它可能是来自商业 CA 的可信证书。
*   由 CA 认证的服务器证书。这将被称为`server.crt`。这是 Mosquitto 将在 TLS 握手期间向客户端提供的证书。
*   对应于服务器证书的主密钥证书。这将被称为`server.key`。

所有这些证书必须是 PEM 格式，并且可以使用`openssl`实用程序轻松生成。

以下示例显示了我用来在默认映像中生成证书的命令。您可能希望使用类似的命令将不同的证书放入映像中；然而，我设想所有这些文件在安装时都会被 OpenShift 覆盖，从 secret 或 configmap(第 2 部分会有更多的介绍)。

```
$ openssl req -new -x509 -days 3650 -extensions v3_ca -keyout files/ca.key \
     -out files/ca.crt -subj "/O=acme/CN=com"
$ openssl genrsa -out files/server.key 2048
$ openssl req -new -out files/server.csr -key files/server.key -subj "/O=acme2/CN=com"
$ openssl x509 -req -in files/server.csr -CA files/ca.crt -CAkey files/ca.key \
    -CAcreateserial -out files/server.crt -days 3650
$ openssl rsa -in files/server.key -out files/server.key
$ rm files/ca.key files/ca.srl files/server.csr
$ chmod 644 files/server.key

```

### 主配置文件

主配置文件指定凭证文件和证书文件的位置，并在端口 1883(明文)和 8883 (TLS)上创建 TCP 侦听器。这些端口号对于 MQTT 来说是常规的，除非另有说明，否则 Mosquitto 实用程序假定它们正在使用中。这里是这个例子的`mosquitto.conf`配置文件。

```
# Port for plaintext communication
port 1883

# Location of the credentials file
password_file /myuser/passwd

# Port and certificates for TLS encrypted communication
listener 8883
certfile /myuser/server.crt
cafile /myuser/ca.crt
keyfile /myuser/server.key
```

注意，所有引用的文件都在`/myuser`中。该目录将在映像构建期间被复制和填充。

实际上，这个配置文件可能需要被覆盖。如果是，新版本可能还需要指定一个凭证文件和证书文件。

### 启动脚本

在这个例子中，启动非常简单。脚本`startup.sh`只是运行`mosquitto`命令，指定主配置文件:

```
#!/bin/sh
mosquitto -c /myuser/mosquitto.conf

```

## 构建、测试和发布映像

为了创建尽可能小的图像，我使用 Alpine Linux 作为基础。Alpine 是专门为容器设计的最小 Linux。为了进一步减小它的大小和复杂性，Alpine 使用了 MUSL C 库，而不是 Linux 中几乎无处不在的 Glibc。这个决定主要是将 Alpine 的使用限制在 Alpine 存储库中可用的应用程序或可以从源代码构建的应用程序。

**注**:在撰写本文时，最新的 Alpine 版本是 3.12。相应的 Alpine 存储库没有 Mosquitto 的二进制包，所以为了简化映像构建，我指定 Alpine 3.11。要使用更高版本的 Alpine，如果存储库中没有这个包，您需要从源代码构建 Mosquitto。构建 Mosquitto 并不难，但是要构建 Alpine 的版本，您必须构建 MUSL C 库。一个简单的方法是安装一个桌面版本的 Alpine Linux，可能是在一个虚拟机中，并在那里进行构建。另一种方法是在映像中构建 Mosquitto，使用多阶段构建过程从最终映像中消除所有构建工具。我的文章 [*在 Red Hat OpenShift*](/blog/2020/08/27/developing-micro-microservices-in-c-on-red-hat-openshift/) 上用 C 开发微微服务中有更多关于这个多阶段构建过程的信息。在这里，我很高兴使用来自 Alpine 存储库的二进制包。

下面是一个简单的 Dockerfile 文件，它使用 Alpine Linux 作为基础来创建映像:

```
FROM alpine:3.11
RUN addgroup -g 1000 mygroup && \
     adduser -G mygroup -u 1000 -h /myuser -D myuser && \
     chown -R myuser:mygroup /myuser && \
     apk --no-cache add mosquitto
WORKDIR /myuser
COPY files/* /myuser/
USER myuser
EXPOSE 1883 8883
CMD ["/myuser/start.sh"]

```

这个文档没有什么特别的。它创建一个用户，公开 Mosquitto 使用的端口，并将我们的启动脚本指定为入口点。

最终图像的总大小只有大约 7 MB。

### 在本地构建和测试映像

使用以下方式构建映像:

```
$ podman build .

```

获取新映像的 ID(例如，使用`podman image list`)然后运行它，指定端口映射。在下面的命令中，用图像的 ID 替换斜体单词*图像*:

```
$ podman run -it -port 1883:1883 -p 8883:8883 *image*
```

此命令公开明文和 TLS 端口。我们可以使用 Mosquitto 测试客户端`mosquitto_pub`(发布)和/或`mosquitto_sub`(订阅)来测试图像。例如，要使用明文侦听器发布消息，请输入:

```
$ mosquitto_pub -t foo -m "Some text" -u admin -P admin

```

`-t foo`选项指定 Mosquitto 主题，而`-m`指定要发送的数据。用户(`-u`选项)和密码(`-P`选项)必须与之前创建凭证文件时在`mosquitto_passwd`命令中给出的值相匹配。注意，主机名和端口默认为`localhost`和 1883，这在本例中是合适的。

要测试 TLS 侦听器，您需要指定 CA 证书。证书在源存储库的`files`目录和映像中(我将在第 2 部分讨论如何从 OpenShift pod 获得证书):

```
$ mosquitto_sub -t foo --cafile files/ca.crt --insecure -u admin -P admin

```

同样，主机和端口的默认值也是合适的。您需要使用`--insecure`开关来覆盖证书主机名检查；这是因为镜像中的服务器证书有主机名`acme.com`，而不是`localhost`。

**注意**:mosquito 默认不耐用。所以如果你想测试消息可以被产生和消费。您需要在启动生产者之前启动消费者。

### 发布图像

如果图像能够在工作站上正常工作，您必须将它发布到一个存储库中，OpenShift 可以从这个存储库中下载它。这样做的过程超出了本文的范围。我已经使用以下`podman`过程将我的图像发布到`quay.io`:

```
$ podman tag <image-id> mosquitto-ephemeral:0.1a
$ podman login quay.io...
$ podman push mosquitto-ephemeral:0.1a quay.io/kboone/mosquitto-ephemeral

```

这是一个公共存储库，所以如果您不想构建自己的存储库，请随意使用我的图像进行测试。我在图像名称中使用标签`ephemeral`来表示这个实现不支持持久消息传递。

当然，您可以将图像发布到另一个公共图像存储库，或者一个私有的机构存储库，或者直接发布到 OpenShift 内部存储库，如果您愿意将它作为一个路径公开的话。详细步骤见我网站上的文章 [*使用 Podman 直接部署镜像到 OpenShift 4*](http://kevinboone.me/podman_deploy.html) 。

无论您如何发布图像，您都需要知道 OpenShift 部署的存储库 URI。在这个使用`quay.io`存储库的例子中，URI 是:

```
quay.io/kboone/mosquitto-ephemeral:latest

```

## 第一部分的结论

本文描述了与在 OpenShift 上部署轻量级应用程序相关的一些设计考虑。我并不认为我的方法是你唯一可以使用的方法，或者甚至是最优的。然而，对于演示来说，这已经足够简单了，并且可能会突出包装者和部署者应该考虑的各种问题。寻找第 2 部分，我们将在 OpenShift 上部署我们在本文中构建的 Mosquitto 图像。

*Last updated: April 15, 2021*
# 红帽 OpenShift 上的 AMQ 在线你好世界

> 原文：<https://developers.redhat.com/products/amq/hello-world-amq-online-openshift>

## 在线安装 AMQ

安装 AMQ 在线有两种选择:下载并解压 YAML 捆绑包的 ZIP 文件，或者使用 OpenShift 容器平台控制台中的 OperatorHub 在 OpenShift 容器平台 4.x 集群上安装 AMQ 在线操作器。在这个快速入门中，我们将使用 YAML 捆绑包。你可以查看 OpenShift 文档上的[在线评估 AMQ，了解如何使用 OperatorHub 在线安装 AMQ 的进一步说明。](https://access.redhat.com/documentation/en-us/red_hat_amq/2020.q4/html/evaluating_amq_online_on_openshift/assembly-getting-started-rh-messaging#proc-olm-installing-from-operatorhub-using-console-messaging)

### 下载并提取

要下载并提取安装文件:

1.  下载 AMQ 在线最新版本的[操作员安装和示例文件](https://access.redhat.com/jbossnetwork/restricted/softwareDownload.html?softwareId=95811)。在写这篇文章的时候，最新的版本是红帽 AMQ 在线 1.7。

2.  将 AMQ 在线安装资源解压缩到任何目的地:

    *   在 Windows 或 Mac 上，通过双击 ZIP 文件提取 ZIP 存档的内容。

    *   在 Red Hat Enterprise Linux (RHEL)上，在目标机器中打开一个终端窗口，并导航到 ZIP 文件的下载位置。通过执行以下命令提取 ZIP 文件:

        `# unzip amq-online-1.7.0-install.zip`

3.  将当前路径更改为解压缩后的目录。

    `# cd amq-online-1.7.0-install`

### 使用 YAML 软件包在线安装 AMQ

要使用 YAML 软件包在线安装 AMQ:

1.  以集群管理员权限登录 OpenShift 集群，例如:

    `# oc login -u system:admin`

2.  默认情况下，安装文件在`amq-online-infra`名称空间中工作。根据您将要安装 AMQ 在线的名称空间修改安装文件，例如`messaging`:

    *   在 Linux 上，使用:

        `# sed -i 's/amq-online-infra/messaging/' install/bundles/amq-online/*.yaml`

    *   在 Mac 上:

        `# sed -i '' 's/amq-online-infra/messaging/' install/bundles/amq-online/*.yaml`

3.  创建您将在线部署 AMQ 的项目，例如`messaging`:

    `# oc new-project messaging`

4.  使用 AMQ 在线捆绑包进行部署:

    `# oc apply -f install/bundles/amq-online`

5.  安装示例计划和基础架构配置:

    `# oc apply -f install/components/example-plans`

6.  安装示例角色:

    `# oc apply -f install/components/example-roles`

7.  安装`standard`认证服务:

    `# oc apply -f install/components/example-authservices/standard-authservice.yaml`

## 创建地址端点

要创建地址端点，您需要创建消息传递地址空间。在 AMQ 在线，您可以使用标准命令行工具创建地址空间。例如:

1.  以普通用户身份登录，如`developer`:

    `# oc login -u developer`

    `# oc new-project my-app-project`

2.  使用`standard small`计划创建类型`standard`的新地址空间(例如`my-address-space`)定义:

    ```
    # cat << EOF | oc create -f -
    apiVersion: enmasse.io/v1beta1
    kind: AddressSpace
    metadata:
      name: my-address-space
    spec:
      type: standard
      plan: standard-small
    EOF
    ```

3.  检查地址空间创建的状态，当配置完成时，它应该返回`true`:

    `# echo `oc get addressspace my-address-space -o jsonpath='{.status.isReady}'``

    这可能需要一些时间。

4.  现在您已经有了一个地址空间，您可以用一个`standard-small-queue`计划创建一个类型为`queue`的名为`my-queue`的地址定义:

    ```
    # cat << EOF | oc create -f -
    apiVersion: enmasse.io/v1beta1
    kind: Address
    metadata:
        name: my-address-space.my-queue
    spec:
        address: my-queue
        type: queue
        plan: standard-small-queue
    EOF
    ```

5.  创建一个消息传递用户，以使用发送和接收权限访问队列:

    ```
    # cat << EOF | oc create -f -
    apiVersion: user.enmasse.io/v1beta1
    kind: MessagingUser
    metadata:
      name: my-address-space.user1
    spec:
      username: user1
      authentication:
        type: password
        password: cGFzc3dvcmQ= # Base64 encoded
      authorization:
        - addresses: ["my-queue"]
          operations: ["send", "recv"]
    EOF
    ```

您现在可以开始发送和接收消息了。

## 发送和接收消息

最后，是时候测试使用外部应用程序发送和接收消息了。您将需要使用 Java 8 来运行以下示例代码。

1.  克隆此 git repo 以测试应用程序对您的新 AMQ 在线地址的访问:

    `$ git clone https://github.com/hguerrero/amq-examples.git`

2.  Switch to the `camel-amqp-demo` folder:

    `$ cd amq-examples/camel-amqp-demo/`

3.  As we are using Routes for external access to the cluster, we need the cluster CA certificate to enable TLS in the client. Extract the public certificate of the broker certification authority:

    `$ oc get addressspace my-address-space -o jsonpath='{.status.caCert}{"\n"}' | base64 --decode > src/main/resources/ca.crt`

4.  Import the trusted cert to a keystore:

    `$ keytool -import -trustcacerts -alias root -file src/main/resources/ca.crt -keystore src/main/resources/truststore.ts -storepass password -noprompt`

5.  Now you can run the [Red Hat Fuse](https://developers.redhat.com/products/fuse/overview) application to send and receive messages to the AMQ Online messaging address endpoint using the following maven command (this command requires Java 8):

    `$ mvn -Drun.jvmArguments="-Damq.url=amqps://`oc get addressspace my-address-space -o jsonpath='{.status.endpointStatuses[?(@.name=="messaging")].externalHost}{"\n"}'`:443 -Damq.destination=queue:my-queue" clean package spring-boot:run`

    完成清理和打包阶段后，您将看到 Spring Boot 应用程序开始创建一个生产者和一个消费者，向“我的队列”AMQ 在线队列地址发送和接收 100 条消息:

    ```
    14:35:52.607 [CamelMainRunController] INFO  o.a.camel.spring.SpringCamelContext - Apache Camel 2.18.1.redhat-000021 (CamelContext: camel) started in 1.368 seconds
    14:35:57.675 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 1
    14:35:57.842 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 2
    14:35:57.854 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 1
    14:35:57.886 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 2
    14:35:57.890 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 3
    14:35:57.928 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 3
    14:35:57.940 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 4
    14:35:57.971 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 4
    14:35:57.992 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 5
    14:35:58.024 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 5
    14:35:58.044 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 6
    14:35:58.080 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 6
    14:35:58.096 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 7
    14:35:58.130 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 7
    14:35:58.147 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 8
    14:35:58.179 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 8
    14:35:58.198 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 9
    14:35:58.230 [Camel (camel) thread #0 - JmsConsumer[my-queue]] INFO  consumer-route - Message received >>> Message 9
    14:35:58.247 [Camel (camel) thread #1 - timer://foo] INFO  producer-route - Sent Message 10

    ```

    你完了！按 Ctrl+C 停止正在运行的程序。

*Last updated: April 1, 2021*
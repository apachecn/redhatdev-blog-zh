# 使用红帽 AMQ 7.7 和 ApacheDS LDAP 服务器进行安全认证

> 原文：<https://developers.redhat.com/blog/2020/08/11/secure-authentication-with-red-hat-amq-7-7-and-apacheds-ldap-server>

在本文中，我们将把[红帽 AMQ 7.7](https://developers.redhat.com/products/amq/overview) 与 [ApacheDS](https://directory.apache.org/apacheds/) LDAP 服务器集成在一起。但是，AMQ 7.x 系列的任何版本都可以与本文中提到的步骤集成。

对于这个集成示例，我们将使用 [Apache Directory Studio](https://directory.apache.org/studio/) ，它是 ApacheDS 的 LDAP 浏览器和目录客户端。您将学习如何从头开始设置 ApacheDS LDAP 服务器，以及如何集成 AMQ 7.7 中所需的新 LDAP 配置更改。最后，我们将使用 [Hawtio](https://hawt.io) 作为图形用户界面(GUI ),测试与 AMQ 7.7 shell 客户端的集成。这对系统管理员和开发人员很有帮助，因为他们可以快速创建 LDAP 和 AMQ 集成的概念证明。这将有助于启用基于角色的访问控制(RBAC)来访问 AMQ 7.7。

**注意**:我们的例子基于[安全-ldap](https://github.com/apache/activemq-artemis/tree/master/examples/features/standard/security-ldap) ，它展示了如何配置和使用带有 ActiveMQ Artemis 和 ApacheDS LDAP 服务器的安全 Java 消息服务(JMS)应用层。此示例随所有 AMQ 7.x 发行版一起提供。我已经测试了 Fedora 32 和 Java 8 的 OpenJDK 版本(1.8.0_252)中的集成。

## 第 1 部分:用 Apache Directory Studio 创建 ApacheDS LDAP 服务器

我们要做的第一件事是使用 Apache Directory Studio 创建一个 ApacheDS 服务器实例:

1.  下载 [Apache Directory Studio](https://directory.apache.org/studio/) 并解压。
2.  一旦它被提取出来，从 Linux 终端执行`ApacheDirectoryStudio/ApacheDirectoryStudio`。
3.  通过选择 **New Server - > Finish** 创建 LDAP 服务器，如图 1 所示。
    [![Dialog to create a new LDAP server.](img/4747e0d4bd309c3fe17f0955dc1d524f.png "Setup New Ldap Server")](/sites/default/files/blog/2020/07/ldap-1-1.png)设置新的 LDAP 服务器

    图 1:在 Apache Directory Studio 中创建新的 Ldap 服务器。

    

### 设置新的 LDAP 服务器

在启动服务器之前，我们可以选择更改它的端口。右键单击服务器并选择**打开配置**。我保留了默认端口。您可以使用 **Ctrl+S** 来保存您所做的任何更改。

[![Dialog to change the LDAP server port.](img/01830ca9128eb9287b3e3bdc575c7c97.png "Change ldap server port.")](/sites/default/files/blog/2020/07/ldap-2-1.png)2\. Change ldap server port.

Figure 2: Change the LDAP server port.

点击该屏幕左下角的**运行**按钮，启动 LDAP 服务器。

### 创建新连接

接下来，我们将创建一个新的连接，如图 3 所示。

[![Dialog to create a new LDAP connection.](img/3da35731cf4a7727099c4d1f6b1cbaf7.png "Create ldap connection")](/sites/default/files/blog/2020/07/ldap-3.png)3\. Create ldap connection

Figure 3: Create a new LDAP connection.

创建连接后，您将在 LDAP 浏览器中看到一个目录树。该树显示在图 4 的左侧面板中。

[![The new directory tree in the LDAP browser window.](img/c5139ee692ae76d6247b2f27ef9fa4d0.png "ldap-4")](/sites/default/files/blog/2020/07/ldap-4.png)

Figure 4: The directory tree in the LDAP browser.

### 创建服务器分区

接下来，我们想要从`security-ldap`示例中导入 [example.ldif](https://github.com/apache/activemq-artemis/blob/master/examples/features/standard/security-ldap/src/main/resources/example.ldif) 文件。在此之前，我们必须创建一个分区。

右键单击 LDAP 服务器并选择 **Open Configuration** 打开 Partitions 选项卡，如图 5 所示。

[![The partitions tab.](img/155d8736cb3bdfcc386c0742ab511012.png "Partition")](/sites/default/files/blog/2020/07/ldap-5.png)5\. Partition

Figure 5: The Partitions tab.

图 6 显示了添加新分区的对话框。输入 **activemq** 作为分区 ID，后缀为 **dc=activemq，dc=org** 。输入 **Ctrl+S** 保存您的更改。

[![Dialog to add a new partition.](img/44d33801df40e0b42e59c07a100aa695.png "Add Partition")](/sites/default/files/blog/2020/07/ldap-6.png)6\. Add Partition

Figure 6: The dialog to add a new partition.

如果你想看到反映的变化，试一下 **F5** 键。

### 导入 example.ldif

现在，我们准备导入`example.ldif`文件，如图 7 所示。

[![Dialog to import the LDIF file.](img/1c74d20c1ba569c759245e1ea6c7994b.png "Import ldif")](/sites/default/files/blog/2020/07/ldap-7.png)7\. Import ldif

图 8 显示了文件的导入配置。

[![Dialog to configure the LDIF connection and import file.](img/ac3cb668590ccac02dacb1ffc4def9f3.png "ldif configuration")](/sites/default/files/blog/2020/07/ldap-8.png)8\. ldif configuration

要完成导入，请停止 LDAP 服务器，然后再次启动它，以便反映更改。重启后，您会看到`example.ldif`文件已经上传，如图 9 所示。

[![A screenshot of the LDIF file in the LDAP browser.](img/b5a9de1db1041f39fa2f60f18a203d10.png "ldif configuration uploaded and reflected.")](/sites/default/files/blog/2020/07/ldap-9.png)9\. ldif configuration uploaded and reflected

### 测试 LDAP 服务器连接

我们几乎完成了 LDAP 服务器的设置。我们的最后一步是测试 LDAP 服务器连接。以下命令请求搜索用户`andrew`:

```
$ ldapsearch -H ldap://localhost:10390 -x -b "uid=andrew,dc=activemq,dc=org"|less

```

输出提供了以下用户详细信息，这也确保了 LDAP 连接正常:

```
# extended LDIF
#
# LDAPv3
# base &amp;lt;uid=andrew,dc=activemq,dc=org&amp;gt; with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# andrew, activemq.org
dn: uid=andrew,dc=activemq,dc=org
objectClass: top
objectClass: simpleSecurityObject
objectClass: account
userPassword:: e1NTSEF9RnQzOGppd3pKVWUwWElsN0VBbm5aQWUxTXJCOWlBUWg0YTRkM2c9PQ=
=
uid: andrew

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

## 第 2 部分:将 AMQ 7.7 与 ApacheDS 集成

在这一节中，我们将把 AMQ 7.7 与 ApacheDS LDAP 服务器集成起来。我还将指导您完成 AMQ 7.7 所需的配置更改。

### 下载并安装 AMQ 7.7

首先，我们将下载并安装 AMQ 7.7。

1.  从[红帽下载入口](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=jboss.amq.broker)下载 AMQ 7.7。或者，你可以下载最新版本的 [Apache Artemis](https://activemq.apache.org/components/artemis/download/) ，这是 AMQ 社区发行版。
2.  解压缩您选择的发行版。在提取的发行版旁边创建一个新文件夹`AMQ_INSTANCE_770`。
3.  将目录更改为`AMQ_INSTANCE_770`，并创建一个名为`brokerInstanceldap` :

    ```
    $ ../amq-broker-7.7.0/bin/artemis create brokerInstanceldap
    Creating ActiveMQ Artemis instance at: /home/chandrashekhar/Development/AMQ_RH/AMQ_INSTANCE_770/brokerInstanceldap

    --user: is a mandatory property!
    Please provide the default username:
    admin

    --password: is mandatory with this configuration:
    Please provide the default password:

    --allow-anonymous | --require-login: is a mandatory property!
    Allow anonymous access?, valid values are Y,N,True,False
    Y

    Auto tuning journal ...
    done! Your system can make 5.68 writes per millisecond, your journal-buffer-timeout will be 176000

    ```

    的 AMQ 代理实例

### 配置新的 AMQ 7.7 实例

您将在本文的 [Git 存储库中找到完整的配置。输入以下内容以克隆此存储库并获取您需要的文件:](https://github.com/1984shekhar/Artemis_POC/tree/master/ldapIntegration)

```
$ git clone https://github.com/1984shekhar/Artemis_POC

```

将这些文件从 Git 存储库中复制到您的`AMQ_INSTANCE_770/brokerInstanceldap/etc`文件夹中:

*   [artemis.profile](https://github.com/1984shekhar/Artemis_POC/blob/master/ldapIntegration/etc/artemis.profile)
*   [broker.xml](https://github.com/1984shekhar/Artemis_POC/blob/master/ldapIntegration/etc/broker.xml)
*   [logging.properties](https://github.com/1984shekhar/Artemis_POC/blob/master/ldapIntegration/etc/logging.properties)
*   [login.config](https://github.com/1984shekhar/Artemis_POC/blob/master/ldapIntegration/etc/login.config)
*   [management.xml](https://github.com/1984shekhar/Artemis_POC/blob/master/ldapIntegration/etc/management.xml)

**注意**:代理配置文件位于`Artemis_POC/ldapIntegration/etc`文件夹中。您也可以在 Git 库中找到它们[。](https://github.com/1984shekhar/Artemis_POC/tree/master/ldapIntegration/etc)

#### 配置详细信息

有关配置的详细信息，请参见以下文件:

*   `login.config`文件包含 LDAP 服务器集成的详细信息。
*   `broker.xml`文件有安全设置。这些设置允许`user`、`europe-user`、`news-user`和`us-user`的角色访问、生成和消费各种队列，这些队列也在文件中列出。
*   `example.ldif`文件保存了 LDAP 服务器角色及其相关用户和密码的定义。

#### 附加配置

除了 LDAP 服务器角色之外，我们还必须定义一个特殊的角色来访问 Hawtio，这是我们在`artemis.profile`配置中完成的。在这种情况下，我们用`-Dhawtio.role=user`替换默认的`amq`。

我们还想为详细日志记录配置 LDAP 服务器操作，因此我们在`logging.properties`文件中设置了以下内容:

```
loggers=[default entries],org.apache.activemq.artemis.spi.core.security
logger.org.apache.activemq.artemis.spi.core.security.level=DEBUG

```

另外，请注意 Hawtio GUI 使用 [Java 管理扩展](https://www.baeldung.com/java-management-extensions) (JMX)框架进行各种操作。因此，我们还必须为 JMX 提供基于角色的访问控制(RBAC)配置。您将在`management.xml`文件中找到以下 RBAC 配置:

```
<match domain="org.apache.activemq.artemis">
            <access method="list*" roles="amq,user"/>
            <access method="get*" roles="amq,user"/>
            <access method="is*" roles="amq,user"/>
            <access method="set*" roles="amq,user"/>
            <access method="*" roles="amq,user"/>
</match>

```

## 第 3 部分:测试 LDAP 与 AMQ 7.7 的集成

我们的最后一步是测试集成。

假设配置文件复制在`Artemis_POC/ldapIntegration/etc`文件夹中，输入以下命令启动代理:

```
$ ./artemis run

```

在不同的 Linux 终端中，使用以下命令发送和接收消息:

```
$ ./artemis producer --user andrew --password activemq1 --destination queue://news.europe.europeTopic --message-count 1
Connection brokerURL = tcp://localhost:61616
Producer ActiveMQQueue[news.europe.europeTopic], thread=0 Started to calculate elapsed time ...

Producer ActiveMQQueue[news.europe.europeTopic], thread=0 Produced: 1 messages
Producer ActiveMQQueue[news.europe.europeTopic], thread=0 Elapsed time in second : 0 s
Producer ActiveMQQueue[news.europe.europeTopic], thread=0 Elapsed time in milli second : 87 milli seconds
$
$ ./artemis consumer --user frank --password activemq2 --destination queue://news.europe.europeTopic --message-count 1
Connection brokerURL = tcp://localhost:61616
Consumer:: filter = null
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 wait until 1 messages are consumed
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 Consumed: 1 messages
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 Elapsed time in second : 0 s
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 Elapsed time in milli second : 27 milli seconds
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 Consumed: 1 messages
Consumer ActiveMQQueue[news.europe.europeTopic], thread=0 Consumer thread finished
$
$ ./artemis producer --user frank --password activemq2 --destination queue://news.europe.europeTopic --message-count 1
Connection brokerURL = tcp://localhost:61616
Producer ActiveMQQueue[news.europe.europeTopic], thread=0 Started to calculate elapsed time ...

javax.jms.JMSSecurityException: AMQ229032: User: frank does not have permission='SEND' on address news.europe.europeTopic
at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:467)
at org.apache.activemq.artemis.core.protocol.core.impl.ChannelImpl.sendBlocking(ChannelImpl.java:361)
at org.apache.activemq.artemis.core.protocol.core.impl.ActiveMQSessionContext.sendFullMessage(ActiveMQSessionContext.java:552)
at org.apache.activemq.artemis.core.client.impl.ClientProducerImpl.sendRegularMessage(ClientProducerImpl.java:296)
at org.apache.activemq.artemis.core.client.impl.ClientProducerImpl.doSend(ClientProducerImpl.java:268)
at org.apache.activemq.artemis.core.client.impl.ClientProducerImpl.send(ClientProducerImpl.java:143)
at org.apache.activemq.artemis.core.client.impl.ClientProducerImpl.send(ClientProducerImpl.java:125)
at org.apache.activemq.artemis.jms.client.ActiveMQMessageProducer.doSendx(ActiveMQMessageProducer.java:483)
at org.apache.activemq.artemis.jms.client.ActiveMQMessageProducer.send(ActiveMQMessageProducer.java:193)
at org.apache.activemq.artemis.cli.commands.messages.ProducerThread.sendMessage(ProducerThread.java:125)
at org.apache.activemq.artemis.cli.commands.messages.ProducerThread.run(ProducerThread.java:91)
Caused by: ActiveMQSecurityException[errorType=SECURITY_EXCEPTION message=AMQ229032: User: frank does not have permission='SEND' on address news.europe.europeTopic]
... 11 more

```

现在，浏览到控制台登录(**http://localhost:8161/console/log in**)，输入用户名 **andrew** 和密码 **activemq1** 。

现在，您应该能够通过 Hawtio GUI 发送消息了，如图 10 所示。

[![A screenshot of the Hawtio user interface with the option to send a message.](img/3571458fa49e4cc490314cf5e59ae69e.png "Hawtio GUI- Send Message Operation")](/sites/default/files/blog/2020/07/ldap-10.png)10\. Hawtio GUI- Send Message Operation

Figure 10: Send a message via the Hawtio GUI.

发送消息验证集成是否完成。

## 结论

我希望您喜欢这篇文章，并且它帮助您更好地理解了使用红帽 AMQ 7.x 和 Apache ActiveMQ Artemis 的 LDAP 认证。

*Last updated: August 10, 2020*
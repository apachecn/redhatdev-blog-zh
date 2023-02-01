# 将阿帕奇 Karaf 上的红帽 Fuse 7 与红帽 AMQ 7 集成

> 原文：<https://developers.redhat.com/articles/2021/06/02/integrate-red-hat-fuse-7-apache-karaf-red-hat-amq-7>

在本文中，我将演示如何在 [Red Hat Fuse 7.8](/products/fuse/overview) 中使用 Apache Camel 应用程序来生成和使用来自 [Red Hat AMQ 7](/products/amq/overview) 或其上游项目 Apache ActiveMQ Artemis 的消息。这种方法使用 AMQ 核心协议 Java 消息服务(JMS)。想从红帽 Fuse 6 和红帽 AMQ 6 迁移到红帽 Fuse 7 和红帽 AMQ 7 的用户也会发现这个教程很有帮助。

## 设置 ActiveMQ Artemis 代理

下面是设置 ActiveMQ Artemis 代理的步骤:

1.  下载[阿帕奇阿尔特弥斯](https://activemq.apache.org/components/artemis/download/)。在写这篇文章的时候，`apache-artemis-2.17.0-bin.zip`是最新的版本。
2.  解压缩这个发行版。与解压缩后的文件夹并行创建一个名为`apache-artemis-2.17-instance`的文件夹，如下例所示:

    ```
     $ ls -ltr|grep artemis
    drwxr-xr-x. 7 chandrashekhar chandrashekhar 4096 Feb 11 12:21 apache-artemis-2.17.0
    -rw-r--r--. 1 chandrashekhar chandrashekhar 63489635 Apr 24 21:23 apache-artemis-2.17.0-bin.zip
    drwxr-xr-x. 2 chandrashekhar chandrashekhar 4096 Apr 24 21:26 apache-artemis-2.17-instance

    $ cd apache-artemis-2.17-instance

    $ ../apache-artemis-2.17.0/bin/artemis create artemis_instance_1 --user admin --password admin
    Creating ActiveMQ Artemis instance at: /mnt/79fece0c-d480-4c54-8268-eaf9ce6ba53d/Development_SSD/AMQ_RH/apache-artemis-2.17-instance/artemis_instance_1

    --allow-anonymous | --require-login: is a mandatory property!
    Allow anonymous access?, valid values are Y,N,True,False
    N

    Auto tuning journal ...
    done! Your system can make 62.5 writes per millisecond, your journal-buffer-timeout will be 16000

    You can now start the broker by executing:

    "/mnt/79fece0c-d480-4c54-8268-eaf9ce6ba53d/Development_SSD/AMQ_RH/apache-artemis-2.17-instance/artemis_instance_1/bin/artemis" run

    Or you can run the broker in the background using:

    "/mnt/79fece0c-d480-4c54-8268-eaf9ce6ba53d/Development_SSD/AMQ_RH/apache-artemis-2.17-instance/artemis_instance_1/bin/artemis-service" start

    $ ls -ltr
    total 4
    drwxrwxr-x. 8 chandrashekhar chandrashekhar 4096 Apr 24 21:37 artemis_instance_1 
    ```

3.  在`apache-artemis-2.17-instance/etc/broker.xml`中添加以下接受者并保存文件。证书位于[示例 GitHub repo](https://github.com/1984shekhar/Fuse7_examples/tree/master/camel_karaf_artemis_core):

    ```
    <acceptors>
    ---
    <acceptor name="netty-ssl-acceptor">tcp://localhost:61621?sslEnabled=true;keyStorePath=/home/chandrashekhar/certificates/activemq.example.keystore;keyStorePassword=activemqexample</acceptor>
    ---
    </acceptors>
    ```

4.  不要忘记修改密钥库路径。然后，从位置`apache-artemis-2.17-instance/bin`:

    ```
    $ ./artemis run

    # we will see logs like

    2021-04-24 21:56:07,978 INFO [org.apache.activemq.artemis.integration.bootstrap] AMQ101000: Starting ActiveMQ Artemis Server
    ----
    ----
    2021-04-24 21:56:08,900 INFO [org.apache.activemq.artemis.core.server] AMQ221020: Started EPOLL Acceptor at localhost:61621 for protocols [CORE,MQTT,AMQP,HORNETQ,STOMP,OPENWIRE]
    ----
    ----
    2021-04-24 21:56:08,920 INFO [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live
    2021-04-24 21:56:08,920 INFO [org.apache.activemq.artemis.core.server] AMQ221001: Apache ActiveMQ Artemis Message Broker version 2.17.0 [0.0.0.0, nodeID=c6090ffc-a519-11eb-a4d5-b0fc366fae1f]
    ```

    启动代理

## 在卡拉夫设置红帽引信

接下来，在 Karaf 上设置红帽保险丝:

1.  [下载红帽保险丝](https://developers.redhat.com/products/fuse/download)并选择 Karaf 安装程序。写这篇文章的时候，红帽 Fuse 7.8 是最新版本。
2.  解压文件并从`bin`文件夹:

    ```
    $ cd fuse-karaf-7.8.0.fuse-780038-redhat-00001/bin
    $./fuse
    ```

    中运行它
3.  您可以使用`mvn clean install`命令来构建应用程序。应用程序代码和证书位于示例 GitHub 库的[中。](https://github.com/1984shekhar/Fuse7_examples/tree/master/camel_karaf_artemis_core)
4.  根据我们的要求修改[骆驼路线](https://github.com/1984shekhar/Fuse7_examples/blob/master/camel_karaf_artemis_core/src/main/resources/OSGI-INF/blueprint/camel-blueprint.xml)。不要忘记更新信任库路径。我们将 Camel 的 Java 消息服务(JMS)组件用于 Artemis 连接工厂(`ActiveMQJMSConnectionFactory`)。
5.  在 Karaf 控制台中，安装以下功能和应用程序:

    ```
    karaf@root()> features:install pax-jms-artemis
    karaf@root()> features:install pax-jms-pool-pooledjms
    karaf@root()> features:install camel-jms
    karaf@root()> install -s mvn:com.mycompany/karafArtemis/1.0.0-SNAPSHOT
    ```

6.  在 Red Hat Fuse 日志中，您会看到以下条目:

    ```
    [chandrashekhar@localhost log]$ tail -f fuse.log
    2021-04-24 22:40:59,731 | INFO | sConsumer[someQueue] | consume_message | 64 - org.apache.camel.camel-core - 2.23.2.fuse-780036-redhat-00001 | message 20210424 10:40
    2021-04-24 22:41:59,729 | INFO | sConsumer[someQueue] | consume_message | 64 - org.apache.camel.camel-core - 2.23.2.fuse-780036-redhat-00001 | message 20210424 10:41
    2021-04-24 22:42:59,734 | INFO | sConsumer[someQueue] | consume_message | 64 - org.apache.camel.camel-core - 2.23.2.fuse-780036-redhat-00001 | message 20210424 10:42
    ```

7.  在 Artemis 中，我们可以从代理实例的`bin`文件夹中检查队列统计信息:

    ```
    $ pwd
    /path/to/apache-artemis-2.17-instance/artemis_instance_1/bin

    $ [chandrashekhar@localhost bin]$ ./artemis queue stat --user admin --password admin
    Connection brokerURL = tcp://localhost:61616
    |NAME |ADDRESS |CONSUMER_COUNT |MESSAGE_COUNT |MESSAGES_ADDED |DELIVERING_COUNT |MESSAGES_ACKED |SCHEDULED_COUNT |ROUTING_TYPE |
    |someQueue |someQueue |1 |0 |18 |0 |18 |0 |ANYCAST |
    [chandrashekhar@localhost bin]$
    ```

## 结论

就是这样！我希望这篇文章对你有帮助。您可以在以下链接中找到有关这些技术的更多详细信息:

*   [阿帕奇 ActiveMQ Artemis](https://activemq.apache.org/components/artemis/])
*   [阿帕奇骆驼](https://camel.apache.org/)
*   [Apache Camel JMS 组件](https://camel.apache.org/components/3.4.x/jms-component.html)

*Last updated: August 15, 2022*
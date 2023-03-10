# OpenShift 上的红帽 AMQ 6.3:设置、连接 SSL 客户端和配置日志记录

> 原文：<https://developers.redhat.com/blog/2019/03/28/red-hat-amq-6-3-on-openshift-set-up-connect-ssl-client-and-configure-logging>

在本文中，我们将讨论如何在 [OpenShift](http://openshift.com/) 上设置[红帽 AMQ](https://developers.redhat.com/products/amq/overview) 6.3。我们还将建立一个外部的基于 Camel 的 SSL 客户机，连接到 AMQ 代理，这是一个纯 T4 Java 多协议消息代理。

通过使用本文中的过程，您可以轻松地在 OpenShift 环境中设置代理，并设置一个基于 Camel 的客户机来快速生成和使用消息。此外，您可以更改日志级别以获得详细的日志，从而更好地理解完整的设置。

我推荐使用源到映像(s2i)方法在 OpenShift 上部署红帽 AMQ 6.x，但是如果您不使用 s2i 方法，本文将帮助您配置日志以获得详细日志。注意，这里使用的红帽 AMQ 形象是短暂的；它不支持持久性。

## A.生成密钥库和信任库

1.首先，我们必须生成密钥库和信任库。

```
[cpandey@cpandey certificate_amq_15Feb]$ keytool -genkey -alias mydomain -keyalg RSA -keystore keystore.jks -keysize 2048
[cpandey@cpandey certificate_amq_15Feb]$ keytool -export -alias mydomain -file mydomain.crt -keystore keystore.jks
[cpandey@cpandey certificate_amq_15Feb]$ keytool -import -trustcacerts -alias mydomain -file mydomain.crt -keystore clientkeystore.jks

```

2.添加`keystore.jks`为秘密。

```
[cpandey@cpandey certificate_amq_15Feb]$ oc login https://openshift.cpandey.lab.pnq2.cee.redhat.com:443 -u username -p password
[cpandey@cpandey certificate_amq_15Feb]$ echo '{"kind": "ServiceAccount", "apiVersion": "v1", "metadata": {"name": "amq-service-account"}}' | oc create -f -
[cpandey@cpandey certificate_amq_15Feb]$ oc policy add-role-to-user view system:serviceaccount:amq-demo:amq-service-account
[cpandey@cpandey certificate_amq_15Feb]$ oc secrets new amq-app-secret keystore.jks
[cpandey@cpandey certificate_amq_15Feb]$ oc secrets add sa/amq-service-account secret/amq-app-secret

```

## B.设置代理

1.OpenShift GUI 可用于代理设置。我使用了目录中的模板`JBoss A-MQ 6.3 (Ephemeral with SSL)`。下面的四个图像显示了我必须为代理设置填写的信息。

[![Image 1](img/739097698f9470a046202ae5f86eafc5.png "BROKER_IMAGE_1")](/sites/default/files/blog/2019/02/BROKER_IMAGE_1.png)Image-1

Image 1

[![Image 2](img/e8aa82e06e6f7044b234f0c656cadcff.png "BROKER_IMAGE_2")](/sites/default/files/blog/2019/02/BROKER_IMAGE_2.png)Image-2

Image 2

[![Image 3](img/d6c02e2c97be07aadc784febc84b25a0.png "BROKER_IMG_3")](/sites/default/files/blog/2019/02/BROKER_IMG_3.png)Image-3

Image 3

[![Image 4](img/ae28cc42b90d9ea9cb87ee4121a72868.png "BROKER_IMAGE_4")](/sites/default/files/blog/2019/02/BROKER_IMAGE_4.png)Image-4

Image 4

2.如果一切顺利，我们将拥有一个已创建并处于运行状态的 pod，服务也将是可见和可访问的。

```
[cpandey@cpandey certificate_amq_15Feb]$ oc get pod|grep broker123
broker123-amq-1-pl8lb 1/1 Running 0 2m

[cpandey@cpandey certificate_amq_15Feb]$ oc get service |grep broker123
broker123-amq-amqp ClusterIP 172.30.187.55 none 5672/TCP 2m
broker123-amq-amqp-ssl ClusterIP 172.30.84.222 none 5671/TCP 2m
broker123-amq-mesh ClusterIP None none 61616/TCP 2m
broker123-amq-mqtt ClusterIP 172.30.172.5 none 1883/TCP 2m
broker123-amq-mqtt-ssl ClusterIP 172.30.245.237 none 8883/TCP 2m
broker123-amq-stomp ClusterIP 172.30.172.21 none 61613/TCP 2m
broker123-amq-stomp-ssl ClusterIP 172.30.177.0 none 61612/TCP 2m
broker123-amq-tcp ClusterIP 172.30.222.183 none 61616/TCP 2m
broker123-amq-tcp-ssl ClusterIP 172.30.215.39 none 61617/TCP 2m

```

## C.设置一个基于 SSL Camel 的客户端

1.要从外部访问代理，需要一个 OpenShift 路由。您可以使用以下命令创建路由:

```
[cpandey@cpandey certificate_amq_15Feb]$ oc create route passthrough --service broker123-amq-tcp-ssl

[cpandey@cpandey certificate_amq_15Feb]$ oc get route|grep broker123
broker123-amq-tcp-ssl broker123-amq-tcp-ssl-amq-demo.apps.cpandey.lab.pnq2.cee.redhat.com broker123-amq-tcp-ssl all passthrough None

```

2.一旦你有了 route，你就可以用我的 [GitHub 代码](https://github.com/1984shekhar/fuse-examples-6.3/blob/master/openshift_transacted_activemq_camel_ssl/src/main/resources/OSGI-INF/blueprint/blueprint.xml)作为客户端来测试 broker。
第一条消息会失败，因为有一个异常被显式抛出，该路由将尝试重新传递消息 6 次。此消息将被放入自动创建的死信队列(DLQ)中。

3.下一步是获取更详细的日志，以便更好地分析运行时问题。总是建议使用 s2i 方法进行配置，所以我创建了一个包含以下内容的`log4j.properties`文件。这里的内容是从代理 pod 的`/opt/amq/conf/`目录的`log4j.properties`文件中复制的，我只修改了配置的前几行，这样就不会影响其他默认日志记录。更多信息，请查看`/opt/amq/conf`处 pod 的默认`log4j.properties`。

```
log4j.rootLogger=DEBUG, logfile, console

# Or for more fine grained debug logging uncomment one of these
log4j.logger.org.apache.activemq=DEBUG
log4j.logger.org.apache.camel=DEBUG

# Console appender
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d | %5p | %m%n
log4j.appender.console.threshold=INFO

# File appender
log4j.appender.logfile=org.apache.log4j.RollingFileAppender
log4j.appender.logfile.file=${activemq.base}/data/activemq.log
log4j.appender.logfile.maxFileSize=1024KB
log4j.appender.logfile.maxBackupIndex=5
log4j.appender.logfile.append=true
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d | %-5p | %m | %c | %t%n
log4j.throwableRenderer=org.apache.log4j.EnhancedThrowableRenderer

###########
# Audit log
###########

log4j.additivity.org.apache.activemq.audit=false
log4j.logger.org.apache.activemq.audit=INFO, audit

log4j.appender.audit=org.apache.log4j.RollingFileAppender
log4j.appender.audit.file=${activemq.base}/data/audit.log
log4j.appender.audit.maxFileSize=1024KB
log4j.appender.audit.maxBackupIndex=5
log4j.appender.audit.append=true
log4j.appender.audit.layout=org.apache.log4j.PatternLayout
log4j.appender.audit.layout.ConversionPattern=%-5p | %m | %t%n

```

## D.将日志级别更改为代理的调试模式

1.创建 **`ConfigMap`** 来加载`log4j.properties`文件。

```
[cpandey@cpandey certificate_amq_15Feb]$ oc create configmap logconfig --from-file=./log4j.properties

```

2.用一个子路径修改部署配置，这样我们只在位置`/opt/amq/conf`挂载`log4j.properties`,其他配置文件保持不变。使用下面显示的命令编辑部署配置，并在部署配置中添加`volumeMounts`和`volume`条目。我们也可以从 OpenShift GUI 控制台编辑部署配置。

[![The mount configuration](img/f265630189e95a2c1442961759cb9ffb.png "Broker_IMG5")](/sites/default/files/blog/2019/02/Broker_IMG5.png)mount-configuration

The mount configuration

就是这样；快乐融合！我希望这篇文章能够帮助您对 OpenShift 上的独立 AMQ 6.x 设置有一个基本的了解，如何设置 SSL 客户机来连接代理，以及如何配置调试级日志配置来进行更详细的日志记录，以便您可以分析运行时问题。此配置也可用于修改其他代理配置文件。

以下是其他红帽 AMQ 的文章，你可能会觉得有帮助:

*   [如何为红帽 AMQ 7 消息代理控制台设置 LDAP 认证](https://developers.redhat.com/blog/2018/09/21/setup-ldap-auth-amq-console/)
*   [在红帽 AMQ 经纪公司设立 RBAC](https://developers.redhat.com/blog/2018/08/06/setting-up-rbac-on-red-hat-amq-broker/)
*   [通过 AMQ 互联扩展 AMQ 7 协议经纪人](https://developers.redhat.com/blog/2018/05/17/scaling-amq-7-brokers-with-amq-interconnect/)
*   [自动化 AMQ 7 高可用性部署](https://developers.redhat.com/blog/2018/04/25/automating-amq-7-high-availability-deployment/)

*Last updated: September 3, 2019*
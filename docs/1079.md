# 在红帽 AMQ 经纪人上设置 RBAC

> 原文：<https://developers.redhat.com/blog/2018/08/06/setting-up-rbac-on-red-hat-amq-broker>

在企业界，尤其是在高度管制的行业中，有一件事是很常见的，那就是职责分离。基于角色的访问控制(RBAC)内置了对职责分离的支持。角色决定了用户可以执行和不可以执行的操作。这篇文章提供了一个如何在红帽 AMQ 之上配置适当的 RBAC 的例子，红帽是一个基于开源的 Apache ActiveMQ Artemis 项目的灵活、高性能的消息平台。

在大多数情况下，Red Hat AMQ 的职责划分可以分为三个主要角色:

1.  管理员角色，将拥有所有权限
2.  应用程序角色，拥有向特定地址发布、使用或生成消息、订阅主题或队列，或者创建和删除地址的权限。
3.  操作角色，通过 web 控制台或支持的协议具有只读权限

为了实现这些角色，Red Hat AMQ 有几个需要配置的安全特性，如以下部分所述。

## AMQ 经纪人认证

开箱即用，AMQ 船舶与 Java 认证和授权服务(JAAS)安全管理器。它提供了一个默认的`PropertiesLogin` JAAS 登录模块，该模块从属性文件(`artemis-users.properties`和`artemis-roles.properties`)中读取用户、密码和角色信息。

因此，要添加用户和角色，我们可以使用这个`artemis`命令:

`// artemis user add --user <username> --password <password> --role <role_comma_seperated>`

例如，要添加三个用户及其角色——一个用户具有管理员角色，一个用户具有应用程序角色，一个用户具有操作角色——我们可以使用如下的`artemis`命令:

```
$ artemis user add --user amqadmin --password amqadmin --role amqadmin
$ artemis user add --user amqapps --password amqapps --role amqapps
$ artemis user add --user amqops --password amqops --role amqops
```

除此之外，红帽 AMQ 还提供其他认证插件。更多信息，请参见[官方文档](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html/using_amq_broker/)。

## AMQ 经纪人授权

AMQ 代理授权策略提供了一种灵活的、基于角色的安全模型，用于根据队列各自的地址对队列应用安全性。例如，诸如发布、消费和生成到地址的消息以及创建和删除地址之类的操作都是现成的。此外，这些策略还支持 AMQP、OpenWire、MQTT、STOMP、HornetQ 和本地 Artemis 核心协议等协议。澄清一下，授权策略并不意味着设置 web 控制台的权限。

要配置权限，我们可以编辑`etc`文件夹中的`broker.xml`文件。默认情况下，每个地址模式有八种不同的权限。因此，要实现上述角色，我们可以像这样使用权限:

```
<security-settings>
  <security-setting match="#">
    <permission type="createNonDurableQueue" roles="amqadmin,amqapps"/>
    <permission type="deleteNonDurableQueue" roles="amqadmin,amqapps"/>
    <permission type="createDurableQueue" roles="amqadmin,amqapps"/>
    <permission type="deleteDurableQueue" roles="amqadmin,amqapps"/>
    <permission type="createAddress" roles="amqadmin,amqapps"/>
    <permission type="deleteAddress" roles="amqadmin,amqapps"/>
    <permission type="consume" roles="amqadmin,amqapps"/>
    <permission type="browse" roles="amqadmin,amqapps,amqops"/>
    <permission type="send" roles="amqadmin,amqapps"/>
    <!-- we need this; otherwise ./artemis data imp wouldn't work -->
    <permission type="manage" roles="amqadmin,amqapps"/>
  </security-setting>
</security-settings>

```

根据上面的例子，只有属于角色`amqadmin`和`amqapps`的用户有权对 AMQ 地址(队列/主题)进行操作(发送/消费/浏览/管理消息),以及创建和删除队列。相比之下，属于`amqops`角色的用户只拥有出于监控目的浏览地址的权限。

## AMQ web 控制台授权

RedHat AMQ 的 web 控制台基于 [Hawtio](http://hawt.io/) ，它使用 [Jolokia](https://jolokia.org/) 读取 JMX 操作。因此，要配置 web 控制台的权限，我们需要设置 JMX 权限。具体来说，可以通过与`broker.xml`文件同一个文件夹中的`management.xml`文件(`etc`文件夹)进行设置。简而言之，要实现上述主要角色，我们可以实现如下内容:

```
<role-access>
  <match domain="org.apache.activemq.artemis" >
    <access method="list*" roles="amqops,amqadmin"/>
    <access method="get*" roles="amqops,amqadmin"/>
    <access method="is*" roles="amqops,amqadmin"/>
    <access method="set*" roles="amqadmin"/>
    <access method="browse*" roles="amqops,amqadmin"/>
    <access method="create*" roles="amqadmin"/>
    <access method="delete*" roles="amqadmin"/>
    <access method="send*" roles="amqadmin"/>
    <access method="*" roles="amqadmin"/>
  </match>
</role-access>

```

综上所述，只有属于`amqadmin`的用户才有完全权限。然而，`amqops`用户拥有使用 web 控制台监控代理的只读权限。类似地，`amqapps`角色无权使用任何 JMX 操作，也无权通过 web 控制台登录。

此外，上面的例子向我们展示了权限的方法设置实际上是 JMX 操作的模式。重要的是要认识到，允许登录 web 控制台的角色是从 Java 系统属性`hawtio.role`中读取的。因此，我们需要配置`etc/artemis.profile`文件，如下例所示:

```
JAVA_ARGS=" -XX:+PrintClassHistogram -XX:+UseG1GC -XX:+AggressiveOpts 
-XX:+UseFastAccessorMethods 
-Xms512M -Xmx2G -Dhawtio.realm=activemq  
-Dhawtio.offline="true" -Dhawtio.role="amqadmin,amqops" 
-Dhawtio.rolePrincipalClasses=org.apache.activemq.artemis.spi.core.security.jaas.RolePrincipal 
-Djolokia.policyLocation=${ARTEMIS_INSTANCE_ETC_URI}jolokia-access.xml 
-Djon.id=amq"

```

在上面的配置示例中，唯一需要更改的是`-Dhawtio.role="amqadmin,amqops"`，它指定了允许登录的角色(以逗号分隔)。

## 结论

通过配置上述功能，您可以在 Red Hat AMQ 上实现适当的 RBAC，以提高安全性并强制执行职责分离。如果你身处一个高度监管的行业，这一点尤为重要。

有关 Red Hat AMQ 代理中用户和角色的更多信息，请参见《使用 AMQ 代理指南*的[用户和角色](https://access.redhat.com/documentation/en-us/red_hat_amq/7.1/html/using_amq_broker/users)章节。*

*Last updated: September 3, 2019*
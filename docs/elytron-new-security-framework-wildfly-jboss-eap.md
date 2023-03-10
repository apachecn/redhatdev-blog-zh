# elytron:wild fly/JBoss EAP 中的新安全框架

> 原文：<https://developers.redhat.com/blog/2018/04/20/elytron-new-security-framework-wildfly-jboss-eap>

[Elytron](https://developer.jboss.org/wiki/WildFlyElytron-ProjectSummary) 是一个新的安全框架，随 [WildFly](http://wildfly.org/) 版本 10 和 Red Hat JBoss Enterprise Application Platform(EAP)7.1 一起发布。这个项目是对[投币箱](http://picketbox.jboss.org)和 JAAS 的完全替代。Elytron 是一个单一的安全框架，可用于保护对服务器的管理访问，以及保护部署在 WildFly 中的应用程序。您仍然可以使用遗留的安全框架，它是 PicketBox，但它是一个不推荐使用的模块；因此，不能保证 PicketBox 会包含在 WildFly 的未来版本中。在本文中，我们将探讨 Elytron 的组件以及如何在 Wildfly 中配置它们。

Elytron 项目涵盖以下内容:

*   SSL/TLS
*   安全凭证存储
*   证明
*   批准

在本文中，我们将通过 Elytron 探索在 WildFly 中使用 SSL/TLS。

这是 Elytron 中 SSL/TLS 的基本架构:

![](img/5af017fcca09c903bdc0866847b827ac.png)

这里的关键属性是`SSLContext`，它还引用了以下组件:

*   **密钥管理器:**密钥管理器保存要使用的密钥库的引用并加载密钥。
*   **Trust-Manager:** 这也保存了 key-store 的引用，基本上用于`trustCertificates`。如果密钥管理器引用的密钥库中存在所有证书，则不需要配置信任管理器。但是，对于出站连接，可以使用信任管理器。
*   **安全域:**这是一个可选参数，但是，如果`SSLContext`配置了对安全域的引用，则客户端证书的验证可以作为身份验证来执行，从而确保在允许连接完全打开之前，为登录分配适当的权限。

`SSLContext`还定义了 SSL 通信的类型(单向/双向)以及允许的协议和密码套件细节。

**配置管理界面和回流子系统使用的 SSL context**

在我们开始在 Elytron 中配置 SSL/TLS 之前，我们应该有一个证书。在本教程中，我们将创建一个自签名证书，以了解 SSL/TLS 如何在 Elytron 中工作。

为了管理证书/密钥库，我在这里使用了 Java 附带的基于 CLI 的实用程序。然而，用户可以使用另一个实用程序来管理证书/密钥库，比如 [Portecle](http://portecle.sourceforge.net/) ，它允许图形化地管理密钥库/证书，并且不需要记住很长的命令行。

首先，使用`keytool`生成密钥库和自签名证书，在操作系统终端命令行中执行如下命令:

```
keytool -genkeypair -alias wildfly -keyalg RSA -sigalg SHA256withRSA -validity 365 -keysize 2048 -keypass jboss@123 -storepass jboss@123 -dname "CN=developer.jboss.org, C=IN" -ext san=dns:developers.redhat.org,dns:developers.wildfly.org -keystore wildfly.jks
```

注意:这只是一个示例，您需要根据您的组织要求更改通用名称(CN)和其他属性，并相应地设置密码。

一旦我们准备好了证书/密钥库，需要执行以下步骤来配置 Elytron 子系统以启用 SSL/TLS。在这里，我将使用 JBoss CLI 演示配置。

*   首先，我们需要通过执行目录`$WildFly_Home/bin`中的 [jboss-cli](https://docs.jboss.org/author/display/WFLY/Command+Line+Interface) 命令来连接到 JBoss CLI**T2。**

*   接下来，用新创建的密钥库在 Elytron 子系统中配置一个`key-store`组件(这里`wildfly.jks`放在`$WildFly_Home/ssl`)。

```
/subsystem=elytron/key-store=wildflyKS:add(type=JKS,path="${jboss.home.dir}/ssl/wildfly.jks",credential-reference={clear-text=jboss@123})
```

*   然后，在 Elytron 子系统中创建一个新的`key-manager`组件，引用上面创建的`key-store`组件。为此，需要执行如下命令:

/subsystem = elytron/key-manager = wildflyKM:add(algorithm = sunx 509，key-store=wildflyKS，credential-reference = { clear-text = JBoss @ 123 })

注意:在创建密钥管理器时，我们需要在这里给出密钥库的密码(例如 jboss@123)。

*   最后，参考上一步中创建的`key-manager`组件配置一个新的`server-ssl-context`:

```
/subsystem=elytron/server-ssl-context=wildlfySSC:add(key-manager=wildflyKM,protocols=[TLSv1.2])
```

要通过 Elytron 启用 SSL/TLS，我们需要执行以下两个命令来配置 under flow`https-listener`并将`ssl-context`映射到 Elytron。默认情况下，`https-listener`配置有 ApplicationRealm 安全领域，默认情况下，ApplicationRealm 在 WildFly 第一次启动时生成自签名证书。您需要执行批处理，因为这两个命令必须同时执行，否则您可以删除 https-listener 并使用 ssl-context **再次添加 https-listener。**

```
batch
/subsystem=undertow/server=default-server/https-listener=https:undefine-attribute(name=security-realm)
/subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=ssl-context,value=wildlfySSC)
run-batch
```

现在，为了让管理界面使用相同的`ssl-context`，我们需要在 JBoss CLI 中执行以下命令，这也将为管理界面启用 SSL:

*   在为管理接口配置`ssl-context`之前，我们需要为通过 SSL/TLS 进行通信的管理 http 接口配置`secure-port`

```
/core-service=management/management-interface=http-interface:write-attribute(name=secure-socket-binding,value=management-https)
```

*   将相同的`ssl-context`映射到管理-http 接口，以启用 SSL/TLS

```
/core-service=management/management-interface=http-interface:write-attribute(name=ssl-context,value=wildflySSC)
```

现在，为了测试您的配置和 SSL/TLS 握手，使用您的浏览器通过 HTTPS 协议发出一个请求。为此，您也可以使用`openssl`命令行实用程序，例如:

```
openssl s_client -connect developers.redhat.com:8443
```

您还可以使用 [SSL 测试工具](https://developers.redhat.com/blog/2017/10/27/ssl-testing-tool)来检查证书以及允许的协议和密码。完成设置后，您可以让您的系统投入生产使用。

Elytron 中有一些早期 JBoss 版本中没有的特性:

*   在 Elytron 子系统中使用的证书过期时，Elytron 会在日志中打印一条警告消息。
*   尽管仍然存在一些挑战，但是在不重新启动/重新加载实例的情况下加载证书密钥库是可能的。
*   Elytron 还提供了检查证书细节的功能。

*Last updated: April 19, 2018*
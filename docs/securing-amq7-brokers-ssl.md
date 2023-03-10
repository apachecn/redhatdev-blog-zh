# 使用 SSL 保护 AMQ7 代理(第 2 部分)

> 原文：<https://developers.redhat.com/blog/2017/12/28/securing-amq7-brokers-ssl>

之前我写过一篇关于用 SSL 保护 AMQ7 路由器的文章。这篇文章将对此进行扩展，解释如何用 SSL 保护 JBoss AMQ7 代理，以及如何用 SSL 连接路由器和代理。

## 代理之间的 SSL

如果您还没有从上一篇文章中收集到您的密钥库和信任库文件，那么您需要按照下面的指导进行收集。如果您已经生成了用于保护路由器安全的文件，也可以使用这些文件。

```
 openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 65000 -out cert.pem
 openssl x509 -text -noout -in cert.pem
 openssl pkcs12 -inkey key.pem -in cert.pem -export -out truststore.p12
 openssl pkcs12 -in truststore.p12 -noout -info
```

您应该得到以下文件:

*   key.pem
*   cert.pem
*   truststore.p12

现在您已经有了适当的文件，您需要编辑您的 broker.xml 文件来使用证书。接受者和连接器都需要编辑。在这个例子中，文件在我的 broker/etc 文件夹中，所以我不需要文件路径。如果您将文件放在其他地方，路径是必需的。

```
<acceptors>
 <acceptor name="artemis">tcp://localhost:61616?sslEnabled=true;keyStorePath=truststore.p12;keyStorePassword=password;enabledProtocols=TLSv1,TLSv1.1,TLSv1.2;trustStorePath=truststore.p12;trustStorePassword=password</acceptor>
</acceptors>
<connectors>
 <connector name="my-connector">tcp://localhost:61616?sslEnabled=true;keyStorePath=truststore.p12;keyStorePassword=password;enabledProtocols=TLSv1,TLSv1.1,TLSv1.2;trustStorePath=truststore.p12;trustStorePassword=password</connector>
</connectors> 
```

如果您现在使用 sslEnabled 启动 2 个代理，您可以看到它们之间的流量是安全的。

## 代理和路由器之间的 SSL

在前一篇文章中，我们在路由器配置中设置了 sslProfile。这里还会用到这个。如果您以前没有添加它，现在就添加。

```
sslProfile {    name: router-ssl
   certFile: /absolute/path/to/cert.pem
   keyFile:/absolute/path/to/key.pem
   password: password }

```

接下来，您将在路由器配置中调整代理的连接器，以使用这个 ssl 配置文件。

```
connector {    name: broker1
   host: localhost
   port: 61616
   role: route-container
   saslMechanisms: ANONYMOUS    sslProfile: router-ssl
verifyHostName: no } 
```

在这一步之后，所有进出代理的东西都是安全的。测试愉快！

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: December 27, 2017*
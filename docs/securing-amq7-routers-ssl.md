# 使用 SSL 保护 AMQ7 路由器

> 原文：<https://developers.redhat.com/blog/2017/11/30/securing-amq7-routers-ssl>

AMQ7 充满了令人激动的新技术和功能。然而，现在路由器和代理都在保护您的拓扑结构，这可能会令人困惑。特别是保护路由器和学习如何使用客户端，对于我们这些习惯使用 jks 文件和纯 jms 的人来说，使用 AMQP 是一个挑战。

## 路由器之间的 SSL

保护路由器之间流量的第一步是获取用于密钥和证书的 pem 文件。这些步骤还将为您提供一个 PKCS12 信任库文件，非常适合用于 AMQP 客户端。虽然这一步可以用 keytool 完成，但我们将使用 openssl。

```
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 65000 -out cert.pem
openssl x509 -text -noout -in cert.pem
openssl pkcs12 -inkey key.pem -in cert.pem -export -out truststore.p12
openssl pkcs12 -in truststore.p12 -noout -info
```

接下来，您需要更新路由器配置。这里我们将使用两台路由器，路由器。a 和 router . b。SSL profile 需要添加到这两个路由器配置文件中。

```
sslProfile {
   name: router-ssl
   certFile: /absolute/path/to/cert.pem
   keyFile:/absolute/path/to/key.pem
   password: password
}
```

然后，您需要在 Router.A 上添加或调整路由器间监听器。

```
listener {
   role: inter-router
   host: 0.0.0.0
   port: 10003
   saslMechanisms: ANONYMOUS
   sslProfile: router-ssl
   authenticatePeer: false
   requireSsl: true
}
```

然后，您需要在路由器上添加或调整连接器。b，它将被用来连接到路由器。

```
connector {
   role: inter-router
   host: 0.0.0.0
   port: 10003
   saslMechanisms: ANONYMOUS
   sslProfile: router-ssl
   verifyHostName: no
}
```

完成后，您应该能够启动两台路由器，然后运行类似下面的命令来查看连接。

```
qdstat -b 0.0.0.0:5672 -c
```

## 到路由器的 SSL

保护路由器之间的流量后，下一步应该关注从客户端到路由器的流量。在路由器上。像这样调整主要听众。

```
listener {
   host: 0.0.0.0
   port: amqp
   saslMechanisms: ANONYMOUS
   authenticatePeer: no
   sslProfile: router-ssl
   requireSsl: true
}
```

然后就可以发送到路由器了。您需要从一个没有 ssl 的客户端开始，比如[https://github . com/Apache/qpid-JMS/tree/master/qpid-JMS-examples](https://github.com/apache/qpid-jms/tree/master/qpid-jms-examples)。然后只需将您的连接 URL 调整为安全的，并使用您的 PKCS12 信任库。

**注意:**由于自签名证书和本地主机的使用，此处 VerifyHost 为 false。

```
amqps://localhost:5672?transport.verifyHost=false&transport.storeType=PKCS12&transport.trustStoreLocation=/absolute/path/to/certificate.p12&transport.trustStorePassword=password
```

现在您的路由器使用 SSL 是安全的！

* * *

**点击此处下载并快速上手[红帽 JBoss AMQ](https://developers.redhat.com/products/amq/download/?intcmp=7016000000124eUAAQ) 。**

*Last updated: November 29, 2017*
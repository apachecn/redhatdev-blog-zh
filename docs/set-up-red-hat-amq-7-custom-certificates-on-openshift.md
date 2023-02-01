# 在 OpenShift 上设置红帽 AMQ 7 自定义证书

> 原文：<https://developers.redhat.com/blog/2019/11/26/set-up-red-hat-amq-7-custom-certificates-on-openshift>

计算机网络上的安全通信是系统最重要的要求之一，但要正确设置却很困难。这个例子展示了如何在[红帽 OpenShift](http://developers.redhat.com/openshift/) 平台上使用自定义 X.509 证书设置[红帽 AMQ 7](https://www.redhat.com/en/technologies/jboss-middleware/amq) 端到端 TLS 加密。

### 先决条件

在继续本示例之前，您需要具备以下条件:

*   OpenShift 集群启动并运行。
*   PEM 格式的自定义 X.509 证书(及其链)。
*   活跃的 [Red Hat 客户门户网站](https://access.redhat.com/)账户。

## 程序

开始之前，让我们定义几个方便的变量:

```
PROJECT="demo"
USER="developer"
BASEURL="https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/74-7.4.0.GA"
```

第一步是登录并创建一个新项目来托管我们的代理:

```
oc login -u $USER -p x
oc new-project $PROJECT
```

然后，我们需要为部署创建一个专用的 ServiceAccount，并添加视图角色:

```
echo '{"kind": "ServiceAccount", "apiVersion": "v1", "metadata": {"name": "amq-service-account"}}' | oc create -f -
oc policy add-role-to-user view system:serviceaccount:$PROJECT:amq-service-account
```

此时，我们应该拥有所有可用的自定义证书文件。最有可能的是，这个签名的自定义证书来自安全团队，还有它的私钥和整个证书链(都是 PEM 格式)。

证书文件由以下内容组成:

*   **rootca.pem** :我们域中的根认证机构(ca)。
*   **interm.pem** :为在特定上下文中签署证书而创建的中间 CA。
*   **server.pem** :最终的服务器证书，可以为单个或多个域颁发(通配符)。
*   **server-prk.pem** :与我们的服务器证书相关联的私钥。

使用这些文件，创建一个服务器密钥库，将其转换为 Java 密钥库(JKS)格式，然后信任用于对其签名的证书链:

```
cat interm.pem rootca.pem > chain.pem
cat server.pem chain.pem > bundle.pem
openssl pkcs12 -export -in bundle.pem -inkey server-prk.pem -out server.p12 -name server -CAfile chain.pem -passout pass:secret
keytool -importkeystore -alias server -srcstoretype PKCS12 -srckeystore server.p12 -srcstorepass secret -destkeystore server.jks -deststorepass secret 
keytool -import -noprompt -trustcacerts -alias chain -file chain.pem -keystore server.jks -storepass secret
```

当服务器密钥库准备就绪时，可以将其导入到一个密码中，该密码还必须添加到前面创建的 ServiceAccount 中:

```
oc create secret generic amq-app-secret --from-file=server.jks
oc secrets add sa/amq-service-account secret/amq-app-secret
```

在这个例子中，我们使用持久的 AMQ 7 SSL 模板，因为我们通常希望我们的消息在代理关闭后仍然存在。让我们创建图像流并下载经纪人的模板:

```
oc login -u system:admin
oc replace --force -f $BASEURL/amq-broker-7-image-streams.yaml
curl -o broker.yaml $BASEURL/templates/amq-broker-74-persistence-ssl.yaml
```

为了能够从 Red Hat 容器注册中心下载代理的图像，我们还需要添加一个身份验证秘密，并将其链接到默认的 ServiceAccount:

```
oc create secret docker-registry registry-auth \
    --docker-server=registry.redhat.io \
    --docker-username=<portal-username> \
    --docker-password=<portal-password>
oc secrets link default registry-auth --for=pull
```

下一步是使用下载的模板部署代理，并将我们的密钥库作为参数传递:

```
oc login -u $USER -p x
oc process -f broker.yaml \
    -p APPLICATION_NAME=$PROJECT \
    -p AMQ_USER=admin \
    -p AMQ_PASSWORD=admin \
    -p AMQ_TRUSTSTORE=server.jks \
    -p AMQ_TRUSTSTORE_PASSWORD=secret \
    -p AMQ_KEYSTORE=server.jks \
    -p AMQ_KEYSTORE_PASSWORD=secret \
    | oc create -f -
```

我们差不多完成了。最后一步是创建一个服务和一个直通路由，向外部世界公开所需的端口。这里我们公开了 AMQP 端口，但是您可以对其他可用的协议做同样的事情:

```
oc create -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  labels:
    application: $PROJECT-amq
  name: broker-amq-amqp-ssl
spec:
  ports:
    - port: 5671
      targetPort: 5671
  selector:
    statefulset.kubernetes.io/pod-name: $PROJECT-amq-0
EOF
oc create route passthrough --service=broker-amq-amqp-ssl
```

## Java 客户端设置

您可以使用首选的 JMS 库来构建客户端，但是您肯定需要一个 JKS 格式的信任库来进行单向 TLS 身份验证:

```
keytool -import -noprompt -file server.pem -alias server -keystore truststore.jks -storepass secret
```

如果您想从 OpenShift 外部访问代理，那么您还需要使用一个类似于下面的 ConnectionFactory URL:

```
amqps://broker-amq-amqp-ssl-demo.192.168.64.53.nip.io:443?transport.verifyHost=false&transport.trustStoreLocation=src/main/resources/truststore.jks&transport.trustStorePassword=secret
```

### 附加注释

您必须将证书的 CN 字段中使用的主机名绑定到 DNS 服务器中群集的 HAProxy IP 地址。如果您使用自制的 CA，那么您还需要信任客户机上的链来访问 Hawtio web 控制台。

*Last updated: July 1, 2020*
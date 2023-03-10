# 在 OpenShift 上设置红帽 AMQ 流自定义证书(更新)

> 原文：<https://developers.redhat.com/blog/2020/04/01/set-up-red-hat-amq-streams-custom-certificates-on-openshift-update>

正如我的前一篇文章的[的“附加说明”部分所预期的，从](https://developers.redhat.com/blog/2019/12/18/set-up-red-hat-amq-streams-custom-certificates-on-openshift/)[红帽 AMQ 流](https://www.redhat.com/en/technologies/jboss-middleware/amq) 1.4 开始，终于可以使用您自己的定制证书来加密 Kafka 客户端和经纪人之间的通信了——而不需要提供 CA 证书。自动生成和管理的内部 ca 仍将保留，但只是为了保护集群间的通信。

用户提供的证书可用于所有启用了 TLS 加密的侦听器，如路由、负载平衡器、入口和节点端口类型。在这个完整的示例中，我们将为单向 TLS 身份验证启用外部路由侦听器。

## 先决条件

在继续之前，您需要具备以下条件:

*   OpenShift 集群启动并运行。
*   PEM 格式的自定义 X.509 证书(带有必需的 San)。
*   活跃的 [Red Hat 客户门户网站](https://access.redhat.com/)账户。
*   [红帽 AMQ 流 1.4.0 安装和示例包](https://access.redhat.com/jbossnetwork/restricted/softwareDetail.html?softwareId=79761&product=jboss.amq.streams&version=1.4.0&downloadType=distributions)。
*   具有`cluster-admin`角色的 OpenShift 用户。

## 程序

开始之前，让我们定义几个方便的变量:

```
$ USER="developer"
$ PROJECT="streams"
$ CA_USER="system:admin"
$ RA_SECRET="reg-auth-secret"
$ CLUSTER="my-cluster"

```

第一步是以`cluster-admin`的身份登录并创建一个新项目。我们需要这个角色，因为我们必须安装*集群操作符(CO)* 所需的*定制资源定义(CRDs)* 。然后，一旦项目准备就绪，我们将授予用户管理项目的完全管理权限:

```
$ oc login -u $CA_USER
$ oc new-project $PROJECT
$ oc adm policy add-role-to-user admin $USER

```

为了能够从 [Red Hat Container Registry](https://registry.access.redhat.com/) 下载图像，我们还需要添加一个认证密码(在这里使用您的凭证):

```
$ oc create secret docker-registry $RA_SECRET \
    --docker-server=registry.redhat.io \
    --docker-username= \
    --docker-password=

```

然后，解压缩安装和示例分发包(名称以`-install-examples.zip`结尾),并用您的名称替换默认项目的名称:

```
$ TMP="/tmp/$PROJECT" && rm -rf $TMP && mkdir -p $TMP
$ unzip -qq amq-streams-1.4.0-ocp-install-examples.zip -d $TMP
$ sed -i -e "s/namespace: .*/namespace: $PROJECT/g" $TMP/install/cluster-operator/*RoleBinding*.yaml

```

现在，我们准备安装所有需要的 CRD 和 Strimzi CO:

```
$ oc apply -f $TMP/install/cluster-operator
$ oc secrets link strimzi-cluster-operator $RA_SECRET --for=pull
$ oc set env deploy/strimzi-cluster-operator STRIMZI_IMAGE_PULL_SECRETS=$RA_SECRET

$ oc set env deploy/strimzi-cluster-operator STRIMZI_NAMESPACE=$PROJECT
$ oc apply -f $install_dir/strimzi-admin
$ oc adm policy add-cluster-role-to-user strimzi-admin $USER

```

### 测试集群创建

在这里，我们创建一个带有主题的小型测试集群，只是为了这个例子(这个集群不适合生产):

```
$ oc create -f - <<EOF
apiVersion: kafka.strimzi.io/v1alpha1
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: "2.3.1"
    replicas: 3
    config:
      log.message.format.version: "2.3"
    logging:
      type: inline
      loggers:
        log4j.logger.kafka.controller: INFO
        log4j.logger.kafka.authorizer.logger: INFO
    listeners:
      plain: {}
      external:
        type: route
    readinessProbe:
      initialDelaySeconds: 30
      timeoutSeconds: 10
    livenessProbe:
      initialDelaySeconds: 30
      timeoutSeconds: 10
    template:
        pod:
          terminationGracePeriodSeconds: 120
    storage:
      type: persistent-claim
      size: "1Gi"
    resources:
      requests:
        cpu: "1000m"
        memory: "2Gi"
      limits:
        cpu: "1000m"
        memory: "2Gi"
    tlsSidecar:
      resources:
        limits:
          cpu: "100m"
          memory: "128Mi"
        requests:
          cpu: "100m"
          memory: "128Mi"
  zookeeper:
    replicas: 3
    readinessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    livenessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    storage:
      type: persistent-claim
      size: "1Gi"
    resources:
      requests:
        cpu: "500m"
        memory: "1Gi"
      limits:
        cpu: "500m"
        memory: "1Gi"
    tlsSidecar:
      resources:
        limits:
          cpu: "100m"
          memory: "128Mi"
        requests:
          cpu: "100m"
          memory: "128Mi"
  entityOperator:
    topicOperator:
      resources:
        limits:
          cpu: "250m"
          memory: "256Mi"
        requests:
          cpu: "250m"
          memory: "256Mi"
    userOperator:
      resources:
        limits:
          cpu: "250m"
          memory: "256Mi"
        requests:
          cpu: "250m"
          memory: "256Mi"
    tlsSidecar:
      resources:
        limits:
          cpu: "100m"
          memory: "128Mi"
        requests:
          cpu: "100m"
          memory: "128Mi"
EOF

```

运行前面的命令后，等待集群启动并运行。

### 自定义证书配置

此时，您应该已经有了以下文件:

*   `rootca.pem` -您的域的根证书颁发机构(CA)(可选)。
*   `intermca.pem` -用于签署子域证书的中间 CA(可选)。
*   `server.pem` -用于外部路由监听器的自定义证书。
*   `server-prk.pem` -自定义证书的私钥。

正如我们将看到的，如果您没有使用自签名证书，那么您可以提供一个包含整个信任链的证书(例如，rootca + intermca + server)。

这里要记住的最重要的一点是，您的自定义证书必须包含正确的主题替代名称(San)。这意味着引导路由有一个条目，每个代理有一个条目。通过查看路由的主机/端口列，您可以很容易地找到它们:

```
$ oc get routes
NAME                         HOST/PORT
my-cluster-kafka-0           my-cluster-kafka-0-amqstr.192.168.64.96.nip.io
my-cluster-kafka-bootstrap    my-cluster-kafka-bootstrap-amqstr.192.168.64.96.nip.io

```

在这种特定环境下，PEM 文件必须具有以下扩展名:

```
$ openssl x509 -inform pem -in server.pem -noout -text
# ...
X509v3 extensions:
  X509v3 Basic Constraints: critical
    CA:FALSE
  X509v3 Key Usage:
    Digital Signature, Key Encipherment
  X509v3 Extended Key Usage:
    TLS Web Server Authentication, TLS Web Client Authentication
  X509v3 Subject Alternative Name:
    DNS:my-cluster-kafka-bootstrap-amqstr.192.168.64.96.nip.io, DNS:my-cluster-kafka-0-amqstr.192.168.64.96.nip.io

```

准备就绪后，我们可以创建/更新将托管我们的自定义证书的密码:

```
$ cat server.pem intermca.pem rootca.pem > fullchain.pem
$ oc create secret generic listener-cert \
    --from-file=server-prk.pem --from-file=fullchain.pem \
    --dry-run -o yaml | oc replace --force -f -

```

最后，我们只需要通过编辑集群定义并等待滚动更新完成来配置外部监听器:

```
$ oc edit kafka $CLUSTER
spec:
  kafka:
    # ...
    listeners:
      plain: {}
      external:
        type: route
        configuration:
          brokerCertChainAndKey:
              secretName: listener-cert
              certificate: fullchain.pem
              key: server-prk.pem

```

### Java 客户端设置

以 Java 密钥库(JKS)格式创建信任库，以便验证 Kafka 代理的身份(单向 TLS 认证)。无论信任链的深度如何，客户端只需信任根 CA 公钥:

```
$ keytool -import -noprompt -trustcacerts -alias rootca -file rootca.pem -keystore client-ts.jks -storepass secret

```

要从 OpenShift 外部访问 Kafka，您还需要使用这个引导 URL:

```
$ echo $(oc get routes $CLUSTER-kafka-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'):443

```

## 附加注释

请记住，自定义证书不是由集群操作者管理的，因此您必须在续订过程中手动更新 OpenShift Secret 和客户端的信任库。如果您在 TLS 或外部侦听器已经使用的秘密中更新 Kafka 侦听器证书，还会启动集群滚动更新。

*Last updated: June 29, 2020*
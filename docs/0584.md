# 在 OpenShift 上设置红帽 AMQ 流自定义证书

> 原文：<https://developers.redhat.com/blog/2019/12/18/set-up-red-hat-amq-streams-custom-certificates-on-openshift>

计算机网络上的安全通信是系统最重要的要求之一，但要正确设置却很困难。这个例子展示了如何在[红帽 OpenShift](http://developers.redhat.com/openshift/) 平台上使用自定义 X.509 CA 证书设置[红帽 AMQ 流](https://developers.redhat.com/blog/category/stream-processing/)端到端 TLS 加密。

## 先决条件

在继续本示例之前，您需要具备以下条件:

*   一个 OpenShift 集群，至少有四个 CPU 和 5GB 内存。
*   PEM 格式的自定义 X.509 CA 证书(及其链)。
*   活跃的 [Red Hat 客户门户网站](https://access.redhat.com/)账户。
*   [红帽 AMQ 流 1.3.0 安装和示例包](https://access.redhat.com/jbossnetwork/restricted/softwareDetail.html?softwareId=74481&product=jboss.amq.streams&version=1.3.0&downloadType=distributions)。
*   具有`cluster-admin`角色的 OpenShift 用户。

## 程序

开始之前，让我们定义几个方便的变量:

```
USER="developer"
PROJECT="streams"
CA_USER="system:admin"
RA_SECRET="reg-auth-secret"
CLUSTER="my-cluster"

```

### 建立新项目

之后的第一步是以`cluster-admin`的身份登录，并创建一个新项目来托管我们的集群。我们需要这个角色，因为我们必须安装*集群操作器(CO)* 所需的*定制资源定义(CRDs)* 。然后，我们授予用户完全的管理权限，让他们在项目准备就绪后管理项目:

```
$ oc login -u $CA_USER
$ oc new-project $PROJECT
$ oc adm policy add-role-to-user admin $USER

```

为了能够从 *Red Hat Container Registry* 下载图像，我们还需要添加一个认证密码(在这里使用您的凭证):

```
$ oc create secret docker-registry $RA_SECRET \
      --docker-server=registry.redhat.io \
      --docker-username=<portal-username> \
      --docker-password=<portal-password>
$ oc secrets link default $RA_SECRET --for=pull

```

然后，解压缩*安装和示例*分发包，并用您的名称替换默认的项目名称:

```
TMP="/tmp/$PROJECT" && rm -rf $TMP && mkdir -p $TMP
$ unzip -qq amq-streams-1.3.0-ocp-install-examples.zip -d $TMP
$ sed -i -e "s/namespace: .*/namespace: $PROJECT/g" $TMP/install/cluster-operator/*RoleBinding*.yaml

```

现在，我们准备安装所有需要的 CRD 和 Strimzi CO:

```
$ oc apply -f $TMP/install/cluster-operator
$ oc secrets link strimzi-cluster-operator $RA_SECRET --for=pull
$ oc set env deploy/strimzi-cluster-operator STRIMZI_IMAGE_PULL_SECRETS=$RA_SECRET

$ oc set env deploy/strimzi-cluster-operator STRIMZI_NAMESPACE=$PROJECT
$ oc apply -f $TMP/install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml
$ oc apply -f $TMP/install/cluster-operator/032-RoleBinding-strimzi-cluster-operator-topic-operator-delegation.yaml
$ oc apply -f $TMP/install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml
$ oc apply -f $TMP/install/strimzi-admin
$ oc adm policy add-cluster-role-to-user strimzi-admin $USER

```

### 配置自定义证书

这些命令完成后，我们可以配置我们的自定义 X.509 CA 证书。我希望您已经有了以下文件:

*   **`rootca.pem`** :本域的根证书颁发机构(CA)(可选)。
*   **`intermca.pem`** :用于在特定上下文中签署证书的中间 CA(可选)。
*   **`myca.pem`** :我们自定义的 CA 证书，配合 Apache Kafka 使用。
*   **`myca-prk.pem`** :自定义 CA 证书的私钥。

链中的所有 CA 都应该配置为 X509v3 基本约束中的 CA。这意味着*您不能使用传统的非 CA 证书来替换自生成的证书*(另请参见末尾的附加注释)。这样做的原因是它用于签署代理间通信的证书。

打印出您的自定义证书后，您应该能够看到以下属性:

```
$ openssl x509 -inform pem -in myca.pem -noout -text
...
X509v3 Basic Constraints: 
    CA:TRUE

```

当您有一个有效的 CA 证书时，创建一个包文件，如下所示:

```
$ cat myca.pem intermca.pem rootca.pem > bundle.pem

```

然后，创建包含自定义 CA 的所有必需的机密和标签。这必须在创建自定义集群之前完成(下一步):

```
$ oc create secret generic $CLUSTER-cluster-ca-cert --from-file=ca.crt=bundle.pem
$ oc label secret $CLUSTER-cluster-ca-cert strimzi.io/kind=Kafka strimzi.io/cluster=$CLUSTER

$ oc create secret generic $CLUSTER-cluster-ca --from-file=ca.key=myca-prk.pem
$ oc label secret $CLUSTER-cluster-ca strimzi.io/kind=Kafka strimzi.io/cluster=$CLUSTER

$ oc create secret generic $CLUSTER-clients-ca-cert --from-file=ca.crt=bundle.pem
$ oc label secret $CLUSTER-clients-ca-cert strimzi.io/kind=Kafka strimzi.io/cluster=$CLUSTER

$ oc create secret generic $CLUSTER-clients-ca --from-file=ca.key=myca-prk.pem
$ oc label secret $CLUSTER-clients-ca strimzi.io/kind=Kafka strimzi.io/cluster=$CLUSTER

```

最后，我们可以部署我们的集群定义。请注意我们如何设置`generateCertificateAuthority`来指示 CO 不要生成自签名 CA，否则会覆盖我们之前的配置。

### 示例:临时集群创建(不用于生产)

为了这个例子，我们在这里创建了一个小的临时集群。*不要使用完全相同的生产设置*:

```
$ oc create -f - <<EOF
apiVersion: kafka.strimzi.io/v1alpha1
kind: Kafka
metadata:
  name: $CLUSTER
spec:
  kafka:
    version: "2.3.0"
    replicas: 1
    config:
      num.partitions: 1
      default.replication.factor: 1
      log.message.format.version: "2.3"
    clusterCa:
      generateCertificateAuthority: false
    clientsCa:
      generateCertificateAuthority: false
    listeners:
      plain: {}
      tls: {}
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
      type: ephemeral
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
    replicas: 1
    readinessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    livenessProbe:
      initialDelaySeconds: 15
      timeoutSeconds: 5
    storage:
      type: ephemeral
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

一旦群集启动并运行，您可能希望检查自定义 CA 是否正确加载:

```
$ oc get pods
$ oc logs strimzi-cluster-operator-<uuid>
$ oc logs $CLUSTER-kafka-0 -c kafka

```

### 设置 Java 客户端

为单向 TLS 身份验证创建并使用 Java 密钥库(JKS)格式的信任库:

```
$ oc extract secret/$CLUSTER-cluster-ca-cert --keys=ca.crt --to=- > ca.pem
keytool -import -noprompt -alias root -file ca.pem -keystore truststore.jks -storepass secret

```

如果您想从 OpenShift 外部访问 Kafka，那么您还需要使用这个引导 URL:

```
$ echo $(oc get routes $CLUSTER-kafka-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'):443

```

## 附加注释

我们已经知道大多数安全团队不会轻易发布 CA 证书。我们正在开发一个[增强功能](https://issues.jboss.org/browse/ENTMQST-1371),为 Kafka 监听器提供使用非 CA 证书的选项，让内部自己生成的 CA 来保护代理间的通信。

请注意，当使用本文中解释的自定义 CA 时，您需要负责证书更新。使用自行生成的证书时，此过程是完全自动化的。在任何情况下，在更新之后，您都必须像前面描述的那样重新创建客户端的信任库。

*Last updated: July 1, 2020*
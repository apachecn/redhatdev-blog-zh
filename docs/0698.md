# 红帽 OpenShift 上的云原生消息传递与 Quarkus 和 AMQ 在线

> 原文：<https://developers.redhat.com/blog/2019/09/04/cloud-native-messaging-on-red-hat-openshift-with-quarkus-and-amq-online>

根据该项目网站的说法，Quarkus 是一个[Kubernetes](https://developers.redhat.com/topics/kubernetes/)-为 GraalVM 和 OpenJDK HotSpot 量身定制的原生 Java 堆栈，由同类最佳的 Java 库和标准精心制作而成。从 0.17.0 版本开始，Quarkus 支持使用高级消息队列协议( [AMQP](http://www.amqp.org/) )，这是一个在应用程序或组织之间传递业务消息的开放标准。

[Red Hat AMQ 在线](https://www.redhat.com/en/about/videos/amq-online-nutshell)是一个基于 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 的机制，用于将消息传递作为托管服务来提供。之前，我们已经看到[如何使用 AMQ 在线提供消息](https://developers.redhat.com/blog/2019/05/17/self-service-messaging-with-red-hat-amq-online-and-gitops/)。在本文中，我们将结合 AMQ 在线和 Quarkus，展示如何使用消息传递领域的两项新技术在 OpenShift 上创建现代消息传递设置。

该指南假设您在 OpenShift 上安装了 AMQ 在线。阅读[安装指南](https://access.redhat.com/documentation/en-us/red_hat_amq/7.3/html/evaluating_amq_online_on_openshift_container_platform/index)了解更多信息。AMQ 在线基于 [EnMasse](https://enmasse.io) 开源项目。

我们将从使用反应式消息传递(一个简单的订单处理系统)创建一个 Quarkus 应用程序开始。它包括一个订单生成器，它以固定的时间间隔将订单发送到消息队列，一个订单处理器，它处理来自消息队列的订单，并提供确认，以便在 HTML 页面中查看。

创建应用程序后，我们将展示如何将消息传递配置注入到应用程序中，并使用 AMQ 在线提供我们需要的消息传递资源。

## quartus 应用程序

我们的 Quarkus 应用程序将在 OpenShift 上运行，并且是对 amqp-quickstart 的修改版本。完整的示例客户端可以在[这里](https://github.com/EnMasseProject/enmasse-example-clients/tree/master/quarkus-example-client)找到。

### 订单生成器

订单生成器每隔 5 秒向“订单”地址发送单调递增的订单标识符。

```
@ApplicationScoped
public class OrderGenerator {

    private int orderId = 1;

    @Outgoing("orders")
    public Flowable<Integer> generate() {
        return Flowable.interval(5, TimeUnit.SECONDS)
        .map(tick -> orderId++);
    }
}
```

### 订单处理器

订单处理器甚至更简单；它只是向“确认”地址返回一个确认 id。

```
@ApplicationScoped
public class OrderProcessor {
    @Incoming("orders")
    @Outgoing("confirmations")
    public Integer process(Integer order) {
        // Confirmation id is twice the order id :)
        return order * 2;
    }
}
```

### 确认资源

确认资源是一个 HTTP 端点，用于列出已经生成的确认。

```
@Path("/confirmations")
public class ConfirmationResource {

    @Inject
    @Stream("confirmations") Publisher<Integer> orders;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }

    @GET
    @Path("/stream")
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public Publisher<Integer> stream() {
        return orders;
    }
}
```

### 配置

我们的应用程序需要一些配置信息，以便连接到 AMQ 在线。Quarkus 连接器配置、AMQP 端点信息和客户端凭证都需要提供。虽然将配置放在一个地方是一个好的做法，但是我们将把事情分成几部分来展示配置 Quarkus 应用程序的选项。

#### 连接器

可以使用应用程序属性文件在编译时提供连接器配置:

```
mp.messaging.outgoing.orders.connector=smallrye-amqp
mp.messaging.incoming.orders.connector=smallrye-amqp
```

为了简单起见，我们将只对“订单”地址使用消息队列。在示例应用程序中,“确认”地址将使用内存中的队列。

#### AMQP endpoint

AMQP 端点主机名和端口信息在编译时是未知的，必须注入。可以在 AMQ 在线创建的配置图中提供端点信息；因此，我们将在应用程序清单中将它们设置为环境变量:

```
spec:
  template:
    spec:
      containers:
      - env:
        - name: AMQP_HOST
          valueFrom:
            configMapKeyRef:
              name: quarkus-config
              key: service.host
        - name: AMQP_PORT
          valueFrom:
            configMapKeyRef:
              name: quarkus-config
              key: service.port.amqp
```

#### 资格证书

我们希望能够使用服务帐户令牌来认证 OpenShift 上的消息传递应用程序。为此，我们需要创建一个定制的 ConfigSource，它从 pod 文件系统中读取身份验证令牌:

```
public class MessagingCredentialsConfigSource implements ConfigSource {
    private static final Set<String> propertyNames;

    static {
        propertyNames = new HashSet<>();
        propertyNames.add("amqp-username");
        propertyNames.add("amqp-password");
    }

    @Override
    public Set<String> getPropertyNames() {
        return propertyNames;
    }

    @Override
    public Map<String, String> getProperties() {
        try {
            Map<String, String> properties = new HashMap<>();
            properties.put("amqp-username", "@@serviceaccount@@");
            properties.put("amqp-password", readTokenFromFile());
            return properties;
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }

    @Override
    public String getValue(String key) {
        if ("amqp-username".equals(key)) {
            return "@@serviceaccount@@";
        }
        if ("amqp-password".equals(key)) {
            try {
                return readTokenFromFile();
            } catch (IOException e) {
                throw new UncheckedIOException(e);
            }
        }
        return null;
    }

    @Override
    public String getName() {
        return "messaging-credentials-config";
    }

    private static String readTokenFromFile() throws IOException {
        return new String(Files.readAllBytes(Paths.get("/var/run/secrets/kubernetes.io/serviceaccount/token")), StandardCharsets.UTF_8);
    }
}
```

### 构建和部署应用程序

要部署示例应用程序，您需要 GraalVM 来执行应用程序的本机编译。按照 [Quarkus 指南](https://quarkus.io/guides/building-native-image-guide.html)中的步骤设置您的环境。

然后，按照以下说明下载源代码，构建并部署示例应用程序:

```
git clone https://github.com/EnMasseProject/enmasse-example-clients
cd enmasse-example-clients/quarkus-example-client
oc new-project myapp
mvn -Pnative -Dfabric8.mode=openshift -Dfabric8.build.strategy=docker package fabric8:build fabric8:resource fabric8:apply
```

该应用程序将被部署，但不会启动，直到我们用我们需要的消息资源配置 AMQ 在线。

## 配置消息传递

剩下的部分是配置我们的应用程序所需的消息传递资源。我们需要创建一个地址空间来提供消息传递端点，一个地址来配置我们的消息传递地址，一个消息传递用户来配置客户端凭证。

### 地址空间

AMQ 在线地址空间是一组共享连接端点以及身份验证和授权策略的地址。创建地址空间时，可以配置如何公开消息传递端点:

```
apiVersion: enmasse.io/v1beta1
kind: AddressSpace
metadata:
  name: quarkus-example
spec:
  type: brokered
  plan: brokered-single-broker
  endpoints:
  - name: messaging
    service: messaging
    exports:
    - name: quarkus-config
      kind: configmap
```

### 地址

从一个地址发送和接收消息。地址具有决定其语义的类型和决定为该地址保留的资源量的计划。地址可以这样定义:

```
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: quarkus-example.orders
spec:
  address: orders
  type: queue
  plan: brokered-queue
```

### 消息用户

为了确保只有受信任的应用程序能够向您的地址发送和接收消息，必须创建一个消息用户。对于在集群上运行的应用程序，您可以使用 OpenShift 服务帐户对客户端进行身份验证。“serviceaccount”用户可以这样定义:

```
apiVersion: user.enmasse.io/v1beta1
kind: MessagingUser
metadata:
  name: quarkus-example.app
spec:
  username: system:serviceaccount:myapp:default
  authentication:
    type: serviceaccount
  authorization:
  - operations: ["send", "recv"]
    addresses: ["orders"]
```

### 应用程序配置权限

要允许 AMQ 在线创建用于注入 AMQP 端点信息的配置映射，我们还需要定义角色和角色绑定:

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: quarkus-config
spec:
  rules:
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    verbs: [ "create" ]
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    resourceNames: [ "quarkus-config" ]
    verbs: [ "get", "update", "patch" ]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: quarkus-config
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: quarkus-config
subjects:
- kind: ServiceAccount
  name: address-space-controller
  namespace: amq-online-infra
```

### 应用配置

您可以直接从示例源应用消息传递配置:

```
cd enmasse-example-clients/quarkus-example-client
oc project myapp
oc apply -f src/main/resources/k8s/addressspace
oc apply -f src/main/resources/k8s/address
```

## 验证应用程序

要验证应用程序是否正在运行，首先确保地址已创建并处于活动状态:

```
until [[ `oc get address quarkus-example.prices -o jsonpath='{.status.phase}'` == "Active" ]]; do echo "Not yet ready"; sleep 5; done
```

然后，检索应用程序路由 URL(在浏览器中打开回显的 URL):

```
echo "http://$(oc get route quarkus-example-client -o jsonpath='{.spec.host}')/prices.html"
```

您现在应该会看到一个根据 AMQ 在线发送和接收的消息定期更新的票证。

## 摘要

在本文中，我们展示了如何编写一个使用 AMQP 进行消息传递的 Quarkus 应用程序，配置该应用程序在 Red Hat OpenShift 上运行，并注入从 AMQ 在线配置派生的应用程序配置。然后，我们创建了为应用程序提供消息传递所需的清单。

*Last updated: September 3, 2019*
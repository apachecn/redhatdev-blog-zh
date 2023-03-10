# 如何在 Open Liberty 20.0.0.3 中为 Kafka 特定的消息属性使用新的 Kafka 客户端 API

> 原文：<https://developers.redhat.com/blog/2020/03/17/how-to-use-the-new-kafka-client-api-for-kafka-specific-message-properties-in-open-liberty-20-0-0-3>

在 Open Liberty 20.0.0.3，你现在可以访问 [Kafka 特有的属性](#kafka)，比如消息键和消息头，而不仅仅是消息有效载荷，就像基本的 MicroProfile 反应式消息传递`Message` API 那样。另外，现在[可以在会话 cookie](#ASCA) 、LTPA 和[JWT](https://developers.redhat.com/cheatsheets/microprofile-jwt/)cookie 以及应用程序定义的 cookie 中设置`SameSite`属性。

**注:**点击查看开放自由 20.0.0.3[修复 bug 列表。](https://github.com/OpenLiberty/open-liberty/issues?utf8=%E2%9C%93&q=label%3Arelease%3A20003+label%3A%22release+bug%22)

如果你对 Open Liberty 即将推出的内容感兴趣，可以看看我们当前的开发版本。其中包括 MicroProfile 的更新特性( [Rest 客户端](https://developers.redhat.com/cheatsheets/microprofile-rest-client/)、度量、健康、容错和配置)，以及具有 Open Liberty 的 GraphQL、自动压缩 HTTP 响应和持久 EJB 定时器。

## 使用 20.0.0.3 运行您的应用

如果你用的是 [Maven](https://openliberty.io//guides/maven-intro.html) ，这里是坐标:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.3</version>
    <type>zip</type>
</dependency>
```

或者，对于 [Gradle](https://openliberty.io//guides/gradle-intro.html) :

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.3,)'
}
```

或者，如果您使用 Docker:

```
FROM open-liberty
```

或者，看看我们的下载页面。

[![Ask a question on Stack Overflow](img/c7ce73089d145211720195aaa79cde52.png)](https://stackoverflow.com/tags/open-liberty)

## 卡夫卡特有的属性

新开放的自由是卡夫卡特有的属性。基本的微概要反应式消息传递`Message` API 不允许用户访问除消息有效负载之外的任何内容。原生 Kafka 客户端 API 允许用户访问 Kafka 特定的消息属性，例如消息密钥和消息头。

### 传入消息

对于传入的消息，我们现在允许用户打开一个`Message`来访问底层的`ConsumerRecord`:

```
>@Incoming("channel1")
public CompletionStage consume(Message message) {
    ConsumerRecord<String, String> cr = (ConsumerRecord<String, String>) message.unwrap(ConsumerRecord.class);
    String key = consumerRecord.key();
    String value = consumerRecord.value();
    String topic = consumerRecord.topic();
    int partition = consumerRecord.partition();
    long timestamp = consumerRecord.timestamp();
    Headers headers = consumerRecord.headers();
    // some more code....
    return CompletableFuture.completedFuture(null);
}
```

### 传出消息

对于传出消息，如果有效负载是一个`ProducerRecord`，那么其中的属性将被传递给 Kafka:

```
>@Outgoing("channel2")
public Message publish() throws UnsupportedEncodingException {
   ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>("myTopic", null, "myKey", "myValue");
   producerRecord.headers().add("HeaderKey", "HeaderValue".getBytes("UTF-8"));
   return Message.of(producerRecord);
}
```

上面的例子假设在通道的微配置文件配置中没有预先明确配置主题。如果主题是预先配置的，那么它将优先，而`ProducerRecord`中的主题将被忽略。

### 使用预配置主题的示例

在这个例子中，使用 MicroProfile Config 将主题预配置为`myOtherTopic`，因此在`ProducerRecord`中设置的主题被忽略。以下是微配置文件的配置属性:

```
>mp.reactive.messaging.channel3.connector=liberty-kafka
mp.reactive.messaging.channel3.topic=myOtherTopic #Overrides value in code
```

以下是反应式消息传递 bean:

```
>@Outgoing("channel3")
public Message<ProducerRecord<K, V>> publish() {
   ProducerRecord pr = new ProducerRecord("myTopic", "myValue");
   return Message.of(pr);
}/pre>
```

## 添加`SameSite` cookie 属性

Open Liberty 现在提供了在会话 cookie、LTPA 和 JWT cookie 以及应用程序定义的 cookie 上设置`SameSite`属性的能力。通过添加一个或多个`server.xml`配置选项来实现`SameSite`属性。servlet `javax.servlet.http.Cookie` API 不提供在 cookie 上设置`SameSite`属性的能力。

如果需要`SameSite`属性，设置它的选项目前仅限于使用`HttpServletResponse.addHeader`和`HttpServletResponse.setHeader`，以及构造`Set-Cookie`头。浏览器使用`SameSite`属性来确定特定的 cookie 是否应该随请求一起发送。`SameSite`属性有三个值:`Lax`、`Strict`和`None`，如表 1 所示。

Table 1: Values for the `SameSite` attribute.

| 价值 | 描述 |
| `SameSite=Strict` | cookie 将只与同站点请求一起发送。例如，只有当 cookie 的站点与地址栏中的站点匹配时，浏览器才会发送 cookie。 |
| `SameSite=Lax` | cookie 将与同站点请求和跨站点顶级导航一起发送。例如，单击一个链接。 |
| `SameSite=None` | cookie 将与同站点和跨站点请求一起发送。例如，浏览器为第三方上下文和嵌入内容发送 cookie。 |

要使用`SameSite` cookie 属性:

`<httpSession cookieSameSite="Disabled|Strict|Lax|None"/>`

默认值为`Disabled`。这意味着不会添加任何`SameSite`属性。

`<webAppSecurity sameSiteCookie="Disabled|Strict|Lax|None"/>`

默认值为`Disabled`。这意味着不会添加任何`SameSite`属性。

```
<httpEndpoint id="defaultHttpEndpoint" 
                httpPort="9080" 
                httpsPort="9443" >
   <samesite lax="cookieOne" strict="cookieTwo" none="cookieThree"/>
</httpEndpoint>
```

`<httpEndpoint/>` `SameSite`配置允许以下列方式使用通配符:

*   独立通配符(*)。所有的 cookie 都有`SameSite=Lax`属性，包括会话和 LTPA/JWT cookie，除非已经设置了`<httpSession/>`和/或`<webAppSecurity/>`配置。例如:

```
     <httpEndpoint id="defaultHttpEndpoint" httpPort="9080" httpsPort="9443" >
        <samesite lax="*" />
     </httpEndpoint>

```

*   在一个或多个 cookie 名称的末尾。下面的代码片段将把下面的 cookie 名称映射到`SameSite`属性:
    *   `cookieOne` → `SameSite=Lax`
    *   `cookieTwo` → `SameSite=Strict`
    *   `cookieThree` → `SameSite=None`

```
  <httpEndpoint id="defaultHttpEndpoint" httpPort="9080" httpsPort="9443" >
        <samesite lax="cookie*" strict="cookieTwo" none="cookieThree"/>
  </httpEndpoint>
```

`<httpSession/>`和`<webAppSecurity/>`配置优先于`<httpEndpoint/>configuration>`。

*   当 cookie 匹配`SameSite=None`配置时，`Secure`属性将自动添加到 cookie 中。

**注意:**`<httpEndpoint/>`配置可应用于任何`Set-Cookie`割台。

关于`SameSite`属性的技术细节可以在下面的 RFC 中找到:[cookie:HTTP 状态管理机制](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03#section-4.1.2.7)。

## 现在尝试在红帽运行时打开自由 20.0.0.3

Open Liberty 是 Red Hat Runtimes 产品的一部分。如果你是 Red Hat Runtimes 的用户，你现在可以尝试 Open Liberty。

*Last updated: January 18, 2022*
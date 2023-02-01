# 开放自由 20.0.0.8 中的 JSON 日志更新

> 原文：<https://developers.redhat.com/blog/2020/08/20/json-logging-updates-in-open-liberty-20-0-0-8>

使用 Open Liberty 20.0.0.8，您现在可以定制 JSON 日志中的 HTTP 访问日志字段。这个特性允许您在 JSON 日志中包含来自`accessLogging logFormat`属性的字段。您也可以将 JSON 日志文件直接写到`system.out`，而不用将其包装在`liberty_message`事件中。

我将介绍这些新功能，并让你开始在[开放自由](https://openliberty.io/about)20.0.0.8 使用它们。要查看修复的 bug 列表，请访问开放自由 20.0.0.8 的 [GitHub 库。](https://github.com/OpenLiberty/open-liberty/issues?q=label%3Arelease%3A20008+label%3A%22release+bug%22+)

## 如何使用 Open Liberty 20.0.0.8 运行您的应用

如果您使用的是 [Maven](https://openliberty.io//guides/maven-intro.html) ，请更新以下坐标以使用 Open Liberty 20.0.0.8 运行您的应用程序:

```
    <dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.8</version>
    <type>zip </type>
    </dependency>

```

如果您使用的是 [Gradle](https://openliberty.io//guides/gradle-intro.html) ，请输入:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.8,)'
}

```

如果您使用 docker，请输入:

```
FROM open-liberty

```

参见[开放自由下载页面](https://openliberty.io/)获取开放自由 20.0.0.8 的可下载档案。

## 定制 JSON 日志中的 HTTP 访问日志字段

在 Open Liberty 中，您可以选择将服务器日志格式化为 basic 或 JSON 格式。当日志为 JSON 格式时，您必须指定要发送到`messages.log`或`console.log`和`standard-out`的源(`message`、`trace`、`accessLog`、`ffdc`或`audit`)。

在 Open Liberty 20.0.0.8 中，我们增加了在 JSON 日志中包含来自`accessLogging logFormat`属性的字段的选项。以前，这些日志中只打印选定的字段。现在，您可以在 JSON 日志中包含其他 NCSA(国家超级计算应用中心)访问日志字段。这项新功能可以让您接收更多符合您需求的信息日志。

### 定制 JSON 访问日志字段

当日志是 JSON 格式时，您可以使用新的`jsonAccessLogFields` logging 属性来指定您希望您的访问日志具有默认的字段集还是基于 HTTP `accessLogging logFormat`属性的自定义集。您可以使用`accessLogging logFormat`属性来定义您想要的日志字段。然后，您可以将日志发送到日志分析工具，如 ELK (Elasticsearch，Logstash，Kibana)堆栈。

例如，您可以指定在 JSON 访问日志中包含用户 ID 和请求时间字段。然后，您可以在 Kibana 中通过用户 ID 过滤这些字段，并逐个用户地跟踪性能。

要接收访问日志，您必须设置属性`accessLogging`或`httpAccessLogging`。例如，您可以在您的`server.xml`中设置以下属性:

```
    <httpEndpoint id="defaultHttpEndpoint" httpPort="9080" httpsPort="9443" host="*">
    <accessLogging logFormat='%R{W} %u %{my_cookie}C %s'/>
    </httpEndpoint>
    <logging messageFormat="json" messageSource="message,accessLog" jsonAccessLogFields="logFormat"/>

```

在`messages.log`文件中，您的访问日志现在将包含在`accessLogging logFormat`属性中指定的四个字段(运行时间、用户 ID、cookie 和响应代码):

```
{
    "type": "liberty_accesslog",
    "host": "192.168.1.15",
    "ibm_userDir": "/you/jennifer.zhen.chengibm.com/libertyGit/open-liberty/dev/build.image/wlp/usr/",
    "ibm_serverName": "defaultServer",
    "ibm_cookie_my_cookie": "example_cookie",
    "ibm_responseCode": 200,
    "ibm_datetime": "2020-06-18T09:30:47.693-0400",
    "ibm_sequence": "1592487047653_0000000000001"
}

```

您还可以通过将以下内容添加到您的`server.xml`中，将这一新功能与 Open Liberty 的`logstashCollector-1.0`功能相集成:

```
    <featureManager>
    <feature>logstashCollector-1.0</feature>
    </featureManager>

    <logstashCollector
    jsonAccessLogFields="logFormat">
    </logstashCollector>

```

### 新访问日志字段的完整列表

下表描述了与相应的`logFormat`令牌一起可用的新字段:

| **字段** | **描述** | **日志格式令牌** |
| ibm_remoteIP | 远程 IP 地址；例如 127.0.0.1。 | %a |
| ibm_bytesSent | 以字节表示的响应大小，不包括标头。 | %b |
| ibm_cookie_{cookiename} | 请求中的 Cookie 值。 | %{cookieName}C 或%C |
| ibm_requestElapsedTime | 请求的运行时间:毫秒精度，微秒精度。 | %D |
| ibm_requestHeader_{headername} | 请求中的标头值。 | %{headerName}i |
| ibm_responseHeader_{headername} | 响应的标头值。 | %{headerName}o |
| ibm_requestFirstLine | 请求的第一行。 | %r |
| ibm_requestStartTime | 请求的开始时间，采用 NCSA 格式。 | %t |
| ibm_accessLogDatetime | 发送到访问日志的消息排队等待记录的时间，采用正常的 NCSA 格式。 | %{t}W |
| ibm_remoteUserID | 根据 WebSphere Application Server 特定的$WSRU 头的远程用户。 | %u |

更多信息参见[开放自由日志](https://openliberty.io/)和[开放自由日志和跟踪配置](https://openliberty.io/)的文档。

## 将预先格式化的 JSON 应用程序日志直接写入`System.out`或`System.err`

在这个版本之前，Open Liberty 将任何写入到`System.out`或`System.err`的预先格式化的 JSON 应用程序日志嵌入到`liberty_message`事件的消息字段中。现在，您可以直接将这些日志写到`System.out`或`System.err`中，而不用将它们包装在`liberty_message`事件中。然后，您可以将日志发送到日志分析工具，如 ELK (Elasticsearch，Logstash，Kibana)堆栈。

下面是这个版本之前的一个预先格式化的 JSON 日志示例:

```
{
    "type":"liberty_message",
    "host":"192.168.0.119",
    "ibm_userDir":"\/you\/yushan.lin@ibm.com\/Documents\/archived-guide-log4j\/finish\/target\/liberty\/wlp\/usr\
    ",
    "ibm_serverName":"log4j.sampleServer",
    "message":"{\n   \"timeMillis\" : 1587666082123,\n
            \"thread\" : \"Default Executor-thread-8\",\n
            \"level\" : \"WARN\",\n
            \"loggerName\" : \"application.servlet.LibertyServlet\",\n
            \"message\" : \"hello liberty servlet warning message!\",\n
            \"endOfBatch\" : false,\n
            \"loggerFqcn\" : \"org.apache.logging.log4j.spi.AbstractLogger\",\n
            \"threadId\" : 53,\n
            \"threadPriority\" : 5\n}\r",
    "ibm_threadId":"00000035",
    "ibm_datetime":"2020-04-23T14:21:22.124-0400",
    "module":"SystemOut",
    "loglevel":"SystemOut",
    "ibm_methodName":"",
    "ibm_className":"",
    "ibm_sequence":"1587666082124_000000000001B",
    "ext_thread":"Default Executor-thread-8”
}

```

您现在可以输出 JSON 应用程序日志，这样它们就不会被包装在`liberty_message`事件中。通过在`server.xml`的记录元素中设置`appsWriteJson="true"`来启用该功能。另一种选择是通过将`bootstrap.properties`设置为:`com.ibm.ws.logging.apps.write.json=true`，从服务器启动时开始设置。

有关这一新记录功能的更多信息，请参见[打开自由记录](https://openliberty.io//docs/ref/config/#logging.html)。

## 现在尝试在红帽运行时打开自由 20.0.0.8

Open Liberty 是 Red Hat Runtimes 产品的一部分，可供 [Red Hat Runtimes 用户](https://access.redhat.com/products/red-hat-runtimes)使用。要了解更多关于将 Open Liberty 应用部署到 OpenShift 的信息，请查看我们的 [Open Liberty 指南:将微服务部署到 OpenShift](https://openliberty.io/guides/cloud-openshift.html) 。

*Last updated: June 29, 2022*
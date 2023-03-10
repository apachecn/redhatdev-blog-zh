# 使用支持 http2 连接的 Camel-under flow 组件

> 原文：<https://developers.redhat.com/blog/2017/12/12/using-camel-undertow-component-supporting-http2-connection>

本文将有助于为*骆驼逆流*组件配置 *http2* 协议支持。

*   骆驼的*逆流*组件使用嵌入式*逆流* web-container 版本*逆流-核心:jar:1.4.21* 。这个版本还支持 *http2* 连接。
*   我用过骆驼版本*2 . 21 . 0-来自上游【https://github.com/apache/camel】T2 的快照*。
*   同样，使用*驼色逆流*组件的 ***卷曲*** 版本为 7.53.1。这个 *curl* 版本支持 *- http* 2 标志，用于发送 *http2* 请求。
*   我还使用了 ***nghttp*** 来测试来自 *linux* 终端的应用程序。但是，这篇文章不是关于 *http2* 的见解。
*   对于 *http2* 细节，我发现文章[【1】](http://undertow.io/blog/2015/04/27/An-in-depth-overview-of-HTTP2.html)[【2】](https://developers.google.com/web/fundamentals/performance/http2/)很有帮助。

1.项目结构如下所示。

```
[csp@dhcppc1 undertow-camel-testing]$ tree
.
├── pom.xml
├── README.md
└── src
    └── main
        └── resources
            └── META-INF
                └── spring
                    └── camel-context.xml
```

2.在 *pom.xml 中，*我们必须设置以下依赖关系。

```
<dependency>
<groupId>org.apache.camel</groupId>
<artifactId>camel-core</artifactId>
<version>${camel.version}</version>
</dependency>
<dependency>
<groupId>org.apache.camel</groupId>
<artifactId>camel-spring</artifactId>
<version>${camel.version}</version>
</dependency>
<dependency>
<groupId>org.apache.camel</groupId>
<artifactId>camel-undertow</artifactId>
<version>${camel.version}</version>
</dependency>
<dependency>
<groupId>org.apache.camel</groupId>
<artifactId>camel-http-common</artifactId>
<version>${camel.version}</version>
</dependency>
```

3.搭配驼色版本:

```
<properties>
<camel.version>2.21.0-SNAPSHOT</camel.version>
</properties>
```

4.现在，人们可以使用基于弹簧的 dsl 和配置 T2 组件。假设我们将这个文件命名为 *camel-context.xml* 。

```
<beans 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://camel.apache.org/schema/spring http://camel.apache.org/schema/spring/camel-spring.xsd">
<camelContext id="cbr-example-context" >
<route id="cbr-route" trace="true">
<from id="_from1" uri="undertow:http://localhost:7766/foo/bar"/>
<setBody id="_setBody1">
<constant&gt;Sending Response&lt;/constant>
</setBody>
<log id="_log5" message="Headers ${in.headers}"/>
<log id="_log5" message="Done processing ${body}"/>
</route>
</camelContext>
<bean class="org.apache.camel.component.undertow.UndertowComponent" id="undertow">
<property name="hostOptions" ref="undertowHostOptions"/>
</bean>
<bean
class="org.apache.camel.component.undertow.UndertowHostOptions" id="undertowHostOptions">
<property name="http2Enabled" value="true"/>
</bean>
</beans>
```

5.指向上面注意 ***http2Enabled*** 对于 **UndertowHostOptions** 类设置为 true，默认设置为 false。该 UndertowHostOptions 然后被引用到 **UndertowComponent** ，该 UndertowHostOptions 然后被用于 **camel** *route* 。
6。我们可以在 *pom.xml* 中使用 *camel-maven-plugin* ，然后我们可以使用 maven 命令 *mvn camel:run 来运行它。*

```
<plugin>
<groupId&>org.apache.camel</groupId>
<artifactId>camel-maven-plugin</artifactId>
<version>${camel.version}</version>
<configuration>
<fileApplicationContextUri>src/main/resources/META-INF/spring/camel-context.xml</fileApplicationContextUri>
</configuration>
</plugin>
```

7.一旦成功运行，应该会在*http://localhost:7766/foo/bar*处公开一个 http 服务。
8。我们可以从 **linux** 终端使用 **curl** 和 **nghttp** 命令对此进行测试。我使用了 ***Fedora 26*** ，其中 **curl** 命令和 *http2* 支持是可用的。对于 ***RHEL7*** ，我使用了 **nghttp** 实用程序来测试 *http2* 连接。9。使用*卷曲*命令。

```
[csp@dhcppc1 undertow-camel-testing]$ curl -v --http2 http://localhost:7766/foo/bar
* Trying ::1...
* TCP_NODELAY set
* connect to ::1 port 7766 failed: Connection refused
* Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 7766 (#0)
> GET /foo/bar HTTP/1.1
> Host: localhost:7766
> User-Agent: curl/7.53.1
> Accept: */*
> Connection: Upgrade, HTTP2-Settings
> Upgrade: h2c
> HTTP2-Settings: AAMAAABkAARAAAAAAAIAAAAA
> 
< HTTP/1.1 101 Switching Protocols
< Connection: Upgrade
< Upgrade: h2c
< Date: Sun, 10 Dec 2017 08:43:58 GMT
* Received 101
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200 
< accept: */*
< http2-settings: AAMAAABkAARAAAAAAAIAAAAA
< breadcrumbid: ID-dhcppc1-1512886066149-0-25
< content-length: 16
< user-agent: curl/7.53.1
< date: Sun, 10 Dec 2017 08:43:58 GMT
< 
* Connection #0 to host localhost left intact
Sending Response
```

10.使用 nghttp 命令。

```
cpandey@cpandey camel-undertow]$ nghttp -v http://localhost:7766/foo/bar
[ERROR] Could not connect to the address ::1
Trying next address 127.0.0.1
[  0.000] Connected
[  0.000] send SETTINGS frame <length=12, flags=0x00, stream_id=0>
          (niv=2)
          [SETTINGS_MAX_CONCURRENT_STREAMS(0x03):100]
          [SETTINGS_INITIAL_WINDOW_SIZE(0x04):65535]
[  0.000] send PRIORITY frame <length=5, flags=0x00, stream_id=3>
          (dep_stream_id=0, weight=201, exclusive=0)
[  0.000] send PRIORITY frame <length=5, flags=0x00, stream_id=5>
          (dep_stream_id=0, weight=101, exclusive=0)
[  0.000] send PRIORITY frame <length=5, flags=0x00, stream_id=7>
          (dep_stream_id=0, weight=1, exclusive=0)
[  0.000] send PRIORITY frame <length=5, flags=0x00, stream_id=9>
          (dep_stream_id=7, weight=1, exclusive=0)
[  0.000] send PRIORITY frame <length=5, flags=0x00, stream_id=11>
          (dep_stream_id=3, weight=1, exclusive=0)
[  0.000] send HEADERS frame <length=45, flags=0x25, stream_id=13>
          ; END_STREAM | END_HEADERS | PRIORITY
          (padlen=0, dep_stream_id=11, weight=16, exclusive=0)
          ; Open new stream
          :method: GET
          :path: /foo/bar
          :scheme: http
          :authority: localhost:7766
          accept: */*
          accept-encoding: gzip, deflate
          user-agent: nghttp2/1.21.1
[  0.001] recv SETTINGS frame <length=18, flags=0x00, stream_id=0>
          [SETTINGS_HEADER_TABLE_SIZE(0x01):4096]
          [SETTINGS_MAX_FRAME_SIZE(0x05):16384]
          [SETTINGS_INITIAL_WINDOW_SIZE(0x04):65535]
[  0.001] send SETTINGS frame <length=0, flags=0x01, stream_id=0>
          ; ACK
          (niv=0)
[  0.001] recv SETTINGS frame <length=0, flags=0x01, stream_id=0>
          ; ACK
          (niv=0)
[  0.003] recv (stream_id=13) :status: 200
[  0.003] recv (stream_id=13) accept: */*
[  0.003] recv (stream_id=13) accept-encoding: gzip, deflate
[  0.003] recv (stream_id=13) breadcrumbid: ID-cpandey-pnq-csb-1512577017865-0-3
[  0.003] recv (stream_id=13, sensitive) content-length: 16
[  0.003] recv (stream_id=13) user-agent: nghttp2/1.21.1
[  0.003] recv (stream_id=13, sensitive) date: Wed, 06 Dec 2017 17:11:51 GMT
[  0.003] recv HEADERS frame <length=88, flags=0x04, stream_id=13>
          ; END_HEADERS
          (padlen=0)
          ; First response header
Sending Response[  0.003] recv DATA frame <length=16, flags=0x01, stream_id=13>
          ; END_STREAM
[  0.003] send GOAWAY frame <length=8, flags=0x00, stream_id=0>
          (last_stream_id=0, error_code=NO_ERROR(0x00), opaque_data(0)=[])
```

#### 1.[http://under tow . io/blog/2015/04/27/An-in-depth-overview-of-http 2 . html](http://undertow.io/blog/2015/04/27/An-in-depth-overview-of-HTTP2.html)
2 .[https://developers . Google . com/web/fundamentals/performance/http 2/](https://developers.google.com/web/fundamentals/performance/http2/)

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: December 11, 2017*
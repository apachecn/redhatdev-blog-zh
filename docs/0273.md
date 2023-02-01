# 开放自由 20.0.0.12 带来了对 gRPC、自定义 JNDI 名字和 Java SE 15 的支持

> 原文：<https://developers.redhat.com/blog/2020/12/02/open-liberty-20-0-0-12-brings-support-for-grpc-custom-jndi-names-and-java-se-15>

开放自由 20.0.0.12 现在支持 gRPC 1.0 和 gRPC 客户端 1.0。这种通用的开源框架是跨数据中心连接远程服务的有效方式。我们还为 Java 命名和目录接口(JNDI)添加了自定义名称支持，使得在您的 Open Liberty 应用程序中查找和注入 Jakarta Enterprise bean(EJB)更加容易。最后，这个新版本与 Java SE 15 兼容，Java SE 15 是最新的 Java 标准版。我们将介绍这些功能，并向您展示如何在 Open Liberty 20.0.0.12 中设置和配置新的 gRPC 和自定义 JNDI 名称支持。

**注**:查看[开放自由 20.0.0.12 问题页面](https://github.com/OpenLiberty/open-liberty/issues?q=label%3A%22release+bug%22+label%3Arelease%3A200012)获取最近修复的错误列表。

## 使用开放自由 20.0.0.12 运行您的应用程序

在 [Maven](https://openliberty.io//guides/maven-intro.html) 中，添加以下坐标更新到 Open Liberty 20.0.0.12:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.12</version>
    <type>zip</type>
</dependency>

```

如果您使用的是[grade](https://openliberty.io//guides/gradle-intro.html)，请输入:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.12,)'
}

```

对于 Docker 来说，它是:

```
FROM open-liberty
```

## gRPC 退出开放自由 20.0.0.12 的测试版

gRPC 框架是一种通用的开源技术，用于有效地连接远程服务。该框架的优势包括出色的性能、简单的服务定义(通过协议缓冲区)、跨平台和跨语言支持，以及广泛的行业采用。您可以从 web 应用程序中提供和使用 gRPC 服务。

开放自由 20.0.0.12 让你比以往更容易使用 gRPC。以前只有测试版才有的两个功能现在已经普遍提供:

*   让您使用 gRPC 服务。
*   `grpcClient-1.0`允许您使用 gRPC 客户端拨打电话。

### 使用 gRPC 服务

`grpc-1.0`特性通过`io.grpc.BindableService`扫描 gRPC 服务实现的 web 应用程序来工作。要符合 gRPC 服务实现的条件，web 应用程序必须包含协议缓冲编译器为其打算提供的服务生成的代码。此外，服务类必须提供无参数构造函数。web 应用程序不需要包含任何核心 gRPC 库；Open Liberty 运行时提供了这些。一旦扫描并启动了 gRPC 服务，远程 gRPC 客户端就可以通过配置的 HTTP 端口访问该服务。

### gRPC 客户端

`grpcClient-1.0`特性为应用程序提供了对 [Netty](https://netty.io/) gRPC 客户端和相关库的访问。web 应用程序必须提供客户端实现和存根，并且可以使用`io.grpc.ManagedChannel`进行出站调用，而不需要提供支持的客户端库。

### gRPC 示例实现

作为一个示例实现，考虑这个基本的 Hello World 服务，其中我们向应用程序的`server.xml`添加了`grpc-1.0`特性:

```
package com.ibm.ws.grpc;

import com.ibm.ws.grpc.beans.GreetingBean;

import io.grpc.examples.helloworld.GreeterGrpc;
import io.grpc.examples.helloworld.HelloReply;
import io.grpc.examples.helloworld.HelloRequest;
import io.grpc.stub.StreamObserver;

public class HelloWorldService extends GreeterGrpc.GreeterImplBase {

    public HelloWorldService(){}

    @Override
    public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
        HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
        responseObserver.onNext(reply);
        responseObserver.onCompleted();
    }
}

```

在这种情况下，应用程序必须提供 [helloworld protobuf 定义](https://github.com/grpc/grpc-java/blob/master/examples/src/main/proto/helloworld.proto)以及`protobuf`编译器输出。我们不需要提供任何额外的库。一旦启动，Hello World 欢迎服务将可以在服务器的 HTTP 端点上访问。

对于一个客户端示例，我们可以使用 gRPC 和`grpcClient-1.0`特性定义一个基本的 servlet:

```
package com.ibm.ws.grpc;

import io.grpc.examples.helloworld.GreeterGrpc;
import io.grpc.examples.helloworld.HelloReply;
import io.grpc.examples.helloworld.HelloRequest;

import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
...
@WebServlet(name = "grpcClient", urlPatterns = { "/grpcClient" }, loadOnStartup = 1)
public class GrpcClientServlet extends HttpServlet {

        ManagedChannel channel;
        private GreeterGrpc.GreeterBlockingStub greetingService;

        private void startService(String address, int port)
        {
            channel = ManagedChannelBuilder.forAddress(address , port).usePlaintext().build();
            greetingService = GreeterGrpc.newBlockingStub(channel);
        }

        private void stopService()
        {
            channel.shutdownNow();
        }

        @Override
        protected void doGet(HttpServletRequest reqest, HttpServletResponse response)
            throws ServletException, IOException
        {

            // set user, address, port params
        }

        @Override
        protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
        {

        // grab user, address, port params
        startService(address, port);
        HelloRequest person = HelloRequest.newBuilder().setName(user).build();
        HelloReply greeting = greetingService.sayHello(person);

        // send the greeting in a response
        stopService();
        }
    }
}

```

与服务示例一样，应用程序必须提供 [helloworld protobuf 定义](https://github.com/grpc/grpc-java/blob/master/examples/src/main/proto/helloworld.proto)和`protobuf`编译器输出。所有必需的 gRPC 客户端库都由`grpcClient-1.0`特性提供。

**注意**:参见 [Open Liberty 示例 gRPC GitHub 库](https://github.com/OpenLiberty/sample-grpc)中的说明和示例，了解更多关于在 Open Liberty 中使用新 gRPC 特性的信息。

## 使用自定义 JNDI 名称来查找或注入 EJB

我们在此版本中增加了对自定义 JNDI 名称的支持，从而增强了 Open Liberty 现有的 EJB 功能。应用程序现在可以配置和使用自定义 JDNI 名称来查找或注入企业 beans。应用程序仍然可以使用传统的默认 JNDI 名称。

在这个增强之前，Open Liberty 只支持使用规范定义的 JNDI 名称:`java:global/<app>/<module>/<bean>!<interface>`以及`java:app`和`java:module`的变体来查找企业 beans。默认情况下，您仍然可以使用遗留的 JNDI 名称，但是您现在可以选择为`ibm-ejb-jar-bnd.xml`文件中的每个 EJB 指定一个自定义名称(或者为 EJB 2 和 EJB 1 模块指定`ibm-ejb-jar-bnd.xmi`)。

除了现有规范要求的名称之外，还提供了新的 JNDI 名称选项。对于 EJB 3 模块，如果您不提供自定义名称，以下默认值将可用:

*   短格式本地接口和 home:
    `ejblocal:<package.qualified.interface>`
*   短格式远程接口和 home:
    `<package.qualified.interface>`
*   长格式本地接口和 home:
    `ejblocal:<component-id>#<package.qualified.interface>`
*   长格式远程接口和 home:
    `ejb/<component-id>#<package.qualified.interface>`

`component-id`默认为:
`<application-name>/<module-jar-name>/<ejb-name>`

### 更轻松地迁移 EJB 应用程序

对企业 beans 的自定义 JNDI 名称支持为来自其他平台(包括 WebSphere traditional)的应用程序提供了更容易的迁移路径。

在 Java EE 6 之前，EJB 规范没有规定企业 beans 所需的 JNDI 名称，因此每个平台都提供了特定于平台的默认名称和自定义绑定文件格式。Open Liberty 只支持规范定义的 JNDI 名称，所以从其他平台移植应用程序通常需要修改代码。

新的自定义 JNDI 名称支持允许您从其他平台迁移应用程序，而无需更改代码。现在，您只需要将特定于平台的绑定文件迁移到新的 Open Liberty 特定于平台的绑定文件格式。在某些情况下，使用新的传统默认名称可以允许应用程序迁移到 Open Liberty，而无需在绑定文件中指定自定义 JNDI 名称。

### 配置自定义绑定

有三个位置可以配置应用程序的自定义绑定。

#### 在 ibm-ejb-jar-bnd.xml 中为 EJB 3 指定自定义绑定

有几种方法可以在`ibm-ejb-jar-bnd.xml`的 EJB JAR 模块或 WAR 模块中为 EJB 3 bean 配置定制绑定。首先，下面是如何为每个接口指定一个绑定:

```
<session name="NoInterceptorBasicStateless">
      <interface class="com.ejbs.InventoryService" binding-name="ejb/Inventory"/>
   </session>

```

下面是如何指定组件 ID(默认长格式绑定的前缀):

```
<session name="AccountServiceBean" component-id="Dept549/AccountProcessor"/>

```

您可以指定一个简单的绑定名称(一个名称用于本地和远程服务):

```
<session name="AccountServiceBean" simple-binding-name="ejb/AccountService"/>

```

您还可以指定本地和远程特定于主目录的绑定名称:

```
 <session name="AccountServiceBean" local-home-binding-name="ejblocal:AccountService"/>
   <session name="AccountServiceBean" remote-home-binding-name="ejb/services/AccountService"/>

```

**注意**:参见 [EJB 3.0 和 EJB 3.1 应用程序绑定概述](https://www.ibm.com/support/knowledgecenter/SSEQTP_9.0.5/com.ibm.websphere.base.doc/ae/cejb_bindingsejbfp.html)了解更多关于传统默认绑定和在`ibm-ejb-jar-bnd.xml`中声明自定义 JNDI 名称的新语法。

#### 在 server.xml 中指定自定义绑定

接下来，看看这个如何在`server.xml`中的 EJB JAR 模块或 WAR 模块中为 EJB 3 bean 配置定制绑定的例子。我们使用`<application>`或`<ejbApplication>`元素，如下所示:

```
<ejbApplication location="EJBTest.jar">
      <ejb-jar-bnd>
         <session name="InventoryServiceBean">
            <interface class="com.ejbs.InventoryService" binding-name="ejb/Inventory"/>
         </session>
      </ejb-jar-bnd>
   </ejbApplication>

```

#### 在 ibm-ejb-jar-bnd.xmi 中为 EJB 1 或 EJB 2 指定定制绑定

最后，下面是如何在`ibm-ejb-jar-bnd.xmi`中为 EJB JAR 模块中的 EJB 1 或 EJB 2 bean 配置定制绑定。请注意，EJB 1 和 EJB 2 提供了适用于远程和本地主目录的单一 JNDI 名称:

```
<ejbBindings xmi:id="BeanBinding_8" jndiName="suite/r6x/base/misc/poollimits/SLCMTTxTimeoutHome">
      <enterpriseBean xmi:type="ejb:Session" href="META-INF/ejb-jar.xml#SLCMTTxTimeout"/>
   </ejbBindings>

```

对于既有远程主目录又有本地主目录的 bean，上面的配置提供了这些自定义绑定:

```
Remote Home : suite/r6x/base/misc/poollimits/SLCMTTxTimeoutHome
   Local Home  : local:suite/r6x/base/misc/poollimits/SLCMTTxTimeoutHome

```

### 配置自定义 JNDI 名称

默认情况下，所有 EJB 要素都启用了对自定义和传统默认 JNDI 名称的支持。这种支持不会干扰现有规范定义的`java:`支持。但是，可以通过`server.xml`中的以下设置禁用新支持:

```
<ejbContainer bindToServerRoot="false"/>

```

也可以只禁用遗留的短格式默认 JNDI 名称支持，其中 bean 是使用接口名称绑定的。在`server.xml`中使用以下设置:

```
   <ejbContainer disableShortDefaultBindings="true"/>

```

因为对自定义 JNDI 名称和传统默认值的新支持提供了替代的 JNDI 名称，所以您现在可以禁用 EJB 规范要求的 JNDI 名称。将以下内容添加到`server.xml`:

```
   <ejbContainer bindToJavaGlobal="false"/>

```

最后，您可以在`open-liberty`中的`<ejbContainer>`元素上添加以下新的配置属性:

```
    <ejbContainer customBindingsOnError="FAIL"/>

```

当多个 beans 绑定到同一个 JNDI 名称时，该属性启用失败的应用程序启动。

## Java SE 15 的 Open Liberty 支持

来自 [AdoptOpenJDK](https://adoptopenjdk.net?variant=openjdk15&jvmVariant=openj9) 、[甲骨文](https://jdk.java.net/15/)或其他 OpenJDK 供应商的任何官方 Java SE 15 版本都将与 Open Liberty 一起工作。虽然 Java SE 15 是目前最新的可用版本，但它不是一个长期受支持的版本。标准支持计划于 2021 年 3 月结束。

请记住，Eclipse OpenJ9 [通常比 Hotspot 提供更快的启动时间](https://openliberty.io//blog/2019/10/30/faster-startup-open-liberty.html)。

此版本中添加的主要功能包括:

*   JEP 379 :谢南多
*   [JEP 377](https://openjdk.java.net/jeps/377) : ZGC
*   [JEP 378](https://openjdk.java.net/jeps/378) :文字块
*   [JEP 384](https://openjdk.java.net/jeps/384) :记录(第二次预览)
*   [JEP 360](https://openjdk.java.net/jeps/360) 密封课(预告)

有关下载 Java 15 版本的更多信息，请参见[AdoptOpenJDK.net](https://adoptopenjdk.net/index.html?variant=openjdk15&jvmVariant=openj9)、[Eclipse.org](https://www.eclipse.org/openj9/)或[OpenJDK.java.net](https://openjdk.java.net/groups/hotspot)。

有关在 Open Liberty 中使用`server.env`文件的更多信息，请参见 Open Liberty 的服务器配置文档的`Configuration Files`部分。

## 尝试在红帽运行时打开自由 20.0.0.12

Open Liberty 是 Red Hat Runtimes 产品的一部分，可供 [Red Hat Runtimes 用户](https://access.redhat.com/products/red-hat-runtimes)使用。要了解更多关于将 Open Liberty 应用部署到 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 的信息，请参见我们的 Open Liberty 指南[将微服务部署到 OpenShift](https://openliberty.io/guides/cloud-openshift.html) 。

*Last updated: February 24, 2022*
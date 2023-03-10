# 整合与红帽移动和 OpenShift 快速调解

> 原文：<https://developers.redhat.com/blog/2018/03/28/intercede-rapid-red-hat-mobile-openshift>

[![](img/421b06e249e56eedb31437eb6464d05e.png)](https://developers.redhat.com/blog/wp-content/uploads/2017/11/Logotype_RH_OpenShiftContainerPlatform_wLogo_CMYK_Black-e1510606829985.jpg) 在 [Red Hat Mobile](https://developers.redhat.com/products/mobileplatform/overview/) 我们了解对灵活产品的需求，这种产品能使我们的客户集成他们构建当前和未来应用所需的工具。我们作为 Kubernetes 项目主要贡献者的地位确保了 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/)为客户和最终用户提供了巨大的灵活性。

Red Hat Mobile 还支持高度灵活地集成一系列第三方服务和产品。在本文中，我们将展示 Red Hat Mobile v4 和 OpenShift v3 如何通过与由[](https://www.intercede.com/)提供的第三方产品集成，使客户能够快速部署和保护他们的移动应用。我们将使用 Intercede 的 RapID 产品为我们的移动应用启用双向 TLS(通常称为客户端证书认证或 CCA)。

本文所述步骤的演示可在此处查看:
<！- ![快速演示](https://www.youtube.com/watch?v=7So2h7bopnU) - >

[https://www.youtube.com/embed/7So2h7bopnU?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/7So2h7bopnU?autoplay=0&start=0&rel=0)

关于求情迅速:

许多组织担心密码不再足够安全，尤其是考虑到全球范围内数据保护和强身份认证立法的不断增加。Intercede 的快速证书可以与应用程序的 HTTPS 服务器集成，以实现双向 TLS。它们的快速 SDK 有助于在设备上收集和管理证书，从而与应用服务器建立有效的 TLS 会话。除了强认证之外，客户端证书还可以用于“签署”数据块，这是区块链类型应用的先决条件。

# 先决条件

我们需要以下软件来执行本文中概述的步骤:

*   安装了 Xcode 的 macOS(稍加修改，该指南也适用于 Android)
*   Node.js v8.x
*   去
*   OpenSSL 的缩写形式
*   OpenShift CLI v3.7.x
*   Docker v17.x

我们还需要访问:

*   一个说情账号与 [快速](http://rapidportal.intercede.com)【rapidportal.intercede.com】启用，部分许可证可用。
*   安装了 Red Hat Mobile v4.x 的 OpenShift v3 实例。

这篇文章假设读者对 Red Hat Mobile、OpenShift 和相关技术有一定程度的了解。

# 概述

将 RapID 添加到我们的 Red Hat Mobile 4.x 应用程序所需的核心步骤如下:

1.  在 HTTP 服务器上配置双向 TLS。
2.  在我们的云应用程序(Node.js 应用程序)和客户端应用程序(移动应用程序)之间放置双向 TLS 服务器。
3.  将 RapID SDK 添加到我们的客户端应用程序中。
4.  更新我们应用程序中的登录流程，以便与 RapID 集成。

为了更好地说明这一点，下面提供了一个由 Intercede 提供的架构图:

[![](img/fafd516ba2aae6af83ee635d49b8d434.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Architecture-Overview.png)

在注册或首次登录到后端的过程中，移动客户端将使用 RapID SDK 向 RapID Certificate Authority 请求一个证书，证书的标识符由我们的应用程序后端提供。如果此证书请求成功，RapID SDK 会将生成的证书存储在设备钥匙串中，并使用指纹或 PIN 对其进行保护。对移动后端的后续 HTTPS 请求将要求提供此证书，以通过服务进行身份验证。

将它与 OpenShift 上的 Red Hat Mobile 放在一起会产生以下架构:

[![](img/7f7b36286e2f7b119cddc454dc5c1a40.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Overall-Architecture.png)

# 在 Red Hat Mobile 上创建应用程序

在本例中，我们将从一个新的应用程序开始，但如果您想将 Intercede 的 RapID 与使用 Red Hat Mobile v4.x 部署的现有移动应用程序集成，这些步骤也是有效的。

要创建新的应用程序，请导航至 Red Hat Mobile Application Platform Studio 的项目页面，然后单击左上角的“新建项目”按钮。在下面的屏幕上，选择“Hello World”项目模板，输入名称，并确保在单击 create 之前选中 Cordova 图标旁边的复选框:

[![](img/9cf11d54fda73c453d4d45f6a08a4077.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-RHMAP-Project-Create.png)

创建过程完成后，确保项目的云应用程序已部署。我们可以从云应用程序视图的 deploy 部分部署应用程序:

[![](img/ea5231e129cc5cfd0bce36a765debed8.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-RHMAP-Cloud-Application-Deploy.png)

## 添加登录路线

现在我们已经在 Red Hat Mobile 中创建了一个项目，让我们为该项目的云应用程序部分添加一个登录路径。下面是我们将要使用的代码:

```
app.post('/auth/login', parser, (req, res, next) => {
  const username = req.body.username;
  const password = req.body.password;

  users.validateCredentials(username, password)
  // Once a user is validated we get their anonymous ID
  // In our case this will be a UUID, but it can be another
  // unique value generated using your own technique
  .then((valid) => {
    if (valid) {
      return users.getAnonymousId(username);
    } else {
      throw new Error('authentication failed');
    }
  })
  // The anonymous ID is an indentifier we'll share with
  // RapID that identifies this user uniquely for us both
  // sends a POST to /rapid/credentials of the RapID server
  .then((anonId) => rapId.requestIdentity(anonId))
  // RapID returns a RequestID that we return to the device
  // The device uses this to retrieve an SSL certificate
  // directly from the RapID service using the mobile SDK
  .then((requestId) => {
    res.json({
      requestId: requestId
    });
  })
  // Internal server error or a login failure. Real world
  // applications would handle this more explicitly to
  // determine the exact error type and respond accordingly
  .catch((e) => next(e));
});

```

这里发生了很多事情，所以让我们一点一点来分析:

1.  *app . post*——在 Node.js express 应用程序中定义一个登录端点。
2.  *users . validate credentials*——验证给定的用户名和密码是否正确。
3.  *users . getanonymousid*——如果用户认证成功(即*有效*为真)，那么我们将为该用户生成一个匿名 Id，或者使用之前生成的与他们的帐户相关联的值。
4.  *rapid . Request identity*——为我们的用户请求一个/identity/。我们将生成的匿名 ID 作为用户的共享标识符传递给 RapID。
5.  *RES . JSON*——将来自身份请求响应的 *requestId* 传递给进行登录调用的移动设备

这个流程在他们的文档中有更详细的解释，所以请务必阅读他们的 [快速文档](https://rapidportal.intercede.com/docs/RapID/) 以获得对整体架构的全面解释。

*快速* 模块中的代码是为了简洁起见，围绕着求情休息端点 *发布/快速/凭证* 的一个抽象。将*用户*模块 视为后端中与用户数据库表或 API 交互的典型抽象。

## 添加一个 API 端点

现在我们在云应用程序中有了一个登录路径，但是我们仍然需要创建一个 API 端点，我们将使用 RapID 来保护它。因为我们从模板创建的应用程序已经在 */hello* 下定义了一个端点，所以让我们修改一下，使它在 *application.js* 文件中如下所示:

```
app.use('/api/hello', require('./lib/hello')())
```

这条路线嵌套在 */api* 路径下。我们这样做是为了让我们可以轻松地使用 RapID 保护我们应用程序的特定部分，而让其他部分暴露出来。出于这篇博文的目的，我们将只保护对 */api* 的请求。

# 获取 Apache 服务器的凭证

Intercede 提供了配置各种 HTTP 服务器的说明，但是在这篇博文中我们将只涉及 Apache HTTPD，因此，我们将展示他们的 Apache HTTPD 配置指南的修改版本。

*注意:这些步骤中生成的所有文件都应该存放在同一个工作目录*

使用下面的 OpenSSL 命令，我们将为我们的服务器生成一个自签名证书。当提示输入密码时，将其留空并按 enter 键。当提示输入“常用名”时，使用 OpenShift UI 中的 *【应用= >路由* 条目但去掉 HTTPS 协议前缀；例如*nodejs-cloudappdevezqll-rhmap-rhmap-development . 127 . 0 . 0 . 1 . nip . io*用于我们的示例应用:

```
$ openssl genrsa -des3 -passout pass:x -out server.pass.key 2048

$ openssl rsa -passin pass:x -in server.pass.key -out server.key

$ rm server.pass.key

$ openssl req -new -key server.key -out server.csr

$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

接下来，我们需要从 RapID 获取可信发行者证书文件。这再简单不过了；只需前往 RapID 门户网站，从“服务器证书”部分下载即可。下载完成后，将其重命名为 *可信-ca.cer* 。

[![](img/eb7d263c57bafedede273707f131935f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-RapID-Portal.png)

# 创建我们的 Apache 服务器映像

我们将使用 Apache HTTPD 作为客户端和 Node.js 云应用程序之间的反向代理。这个反向代理是 Intercede 快速证书将被部署的层，以确保客户端在访问我们的 API 端点时提供他们自己的证书来验证他们的身份。

由于我们在 OpenShift 上部署这个应用程序，我们需要在一个容器中运行 Apache，所以让我们从创建一个 docker 文件开始。docker 文件为 docker 提供了如何建立我们的形象的说明。

下面是我们的 docker 文件的样子:

```
FROM registry.access.redhat.com/rhscl/httpd-24-rhel7

# Don't want to see the default welcome page...
RUN rm /etc/httpd/conf.d/welcome.conf 

# These can be generated using OpenSSL
# Details here: rapidportal.intercede.com/docs/RapID/reference_apache/
COPY server.crt /opt/app-root/ssl/server.crt
COPY server.key /opt/app-root/ssl/server.key
# You must download this from the RapID Portal
COPY trusted-ca.cer /opt/app-root/ssl/trusted-ca.cer

# Use our custom config instead of the default one
COPY conf/httpd.conf /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf

# Use our own www content instead of the defaults
COPY ./www /opt/rh/httpd24/root/var/www/html

# Inform docker that we'll be exposing services on these ports
EXPOSE 8080
EXPOSE 8443

# The service name (host) and port of our node.js application
# These can be overwritten from the OpenShift Console or during the creation

ENV RHMAP_HOST 'http://nodejs-cloudappdevezqll.rhmap-rhmap-development.svc'
ENV RHMAP_PORT '8001'
```

*FROM*语句告诉 docker，我们希望使用 Red Hat 的官方 httpd 映像创建一个基于 Red Hat Enterprise Linux (RHEL)的映像。 *COPY* 语句指示 docker 将我们的证书、密钥和*www*内容复制到结果图像中的指定路径——当我们创建 Apache `httpd.conf `配置文件时，我们将再次看到这些内容。*EXPOSE*语句通知 docker 服务将在这些端口上公开。

我们完整的 *httpd.conf* 文件可以在 [这里](https://gist.github.com/evanshortiss/b097a03a906be8a5100ab53896efb695) 找到，但是我们会在下面的段落中指出重要的部分。

我们需要使用以下代码行配置服务器，分别监听 HTTPS 和 HTTP 的 8443 和 8080:

```
Listen 8080
Listen 8443
```

接下来，我们需要定义 TLS/SSL 连接的设置。为此，我们将以下配置放在“httpd . conf ”.的末尾

```
<IfModule ssl_module>
 <VirtualHost _default_:8443>
 ServerAdmin evan.shortiss@redhat.com
 SSLEngine on
 SSLCertificateFile /opt/app-root/ssl/server.crt
 SSLCertificateKeyFile  /opt/app-root/ssl/server.key
 SSLCACertificateFile /opt/app-root/ssl/trusted-ca.cer
 SSLCertificateChainFile /opt/app-root/ssl/trusted-ca.cer

 # Any inbound requests to /api will be authenticated using two way TLS/SSL 
 <Location /api/*>
 AllowMethods GET POST OPTIONS
 SSLVerifyClient require
 SSLOptions +ExportCertData +StdEnvVars +OptRenegotiate
 RequestHeader set SSL_CLIENT_CERT "%{SSL_CLIENT_CERT}s"
 ProxyPass ${RHMAP_HOST}:${RHMAP_PORT}
 </Location>

 # All other traffic will be forwarded to the application
 <Location />
 AllowMethods GET POST OPTIONS
 ProxyPass ${RHMAP_HOST}:${RHMAP_PORT}/
 </Location>
 </VirtualHost>
</IfModule>
```

```
下面是它所引用的文件路径以及它们是什么的简要说明:
1.  server . CRT——我们用 OpenSSL 生成的证书
2.  server . Key——我们用 OpenSSL 生成的密钥
3.  trusted-ca.cer -我们从 RapID 门户网站下载的可信证书颁发机构文件
以下配置规定，对嵌套在 *  /api  * 端点下的路由的任何传入请求将使用双向 TLS 来保护；例如*【http://my-route.openshift.com/api/orders】*使用双向 TLS 进行保护。如果客户端在访问这些路由时没有提供有效的证书，则请求将被拒绝，并出现 TLS 错误。如果提供了有效的证书，那么该请求将被代理到我们的内部服务(云应用程序)http://nodejs-cloudappdevezqll . rhmap-rhmap-development . SVC:8001。
最后，创建一个 *  www/  * 文件夹，并在里面放置一个*【index.html】*，内容如下:

```
<!doctype html>
<html>
 <title>RapID Sample Page</title>

 <body>
 <h2>RapID Sample Page</h2>
 <p>If you're seeing this then the RapID sample server is configured correctly</p>
 </body>
</html>
```

我们可以通过运行以下命令来测试我们的映像:

```
$ docker build -t rapid-proxy .
$ docker run -dit --name rapid-proxy -e RHMAP_HOST=http://nodejs-cloudappdevezqll.rhmap-rhmap-development.svc -e RHMAP_PORT='8001' -v "$PWD/www":/usr/local/apache2/htdocs/ -p 8443:8443 -p 8080:80 rapid-proxy
```

导航到*http://localhost:8080*或*https://localhost:8443*应该会加载我们刚刚创建的*index.html*，但是导航到*https://localhost:8443/API/test*会失败，并出现 SSL 错误。我们可以看到 Google Chrome 报告说服务器需要一个有效的证书，但是我们无法提供:
[![](img/c613f630751595a574dec1c2437fd90a.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Blocked-Request.png)
如果服务器没有如所描述的那样运行，那么获取之前由*docker run*命令打印的 ID，并将其传递给 *  docker 日志 * 以检查错误。发现错误来源后，进行必要的编辑，使用以下命令停止并删除图像，然后再次运行之前的 docker*build*和*run*命令:

```
$ docker stop rapid-proxy
$ docker rm rapid-proxy
$ docker rmi rapid-proxy
```

在 OpenShift 上部署我们的映像
为了本文的目的，我们假设 OpenShift 实例上的容器注册中心可以通过公共路径访问；我们拥有创建图像流所需的权限；我们有权限将图像发送到 OpenShift 的内部注册表。
使用 OpenShift CLI 登录，根据内部映像注册表进行身份验证，标记映像，并使用以下命令将映像标记推送到我们的注册表:

```
$ oc login $OSD_URL

# Set this to your OSD username
$ OC_USER=evanshortiss
# Get a login token to access the docker registry on OpenShift
$ TOKEN=$(oc whoami -t)

# The name of the project where our app is deployed
$ PROJECT_ID=rhmap-rhmap-development

# Get our docker registry url
$ REGISTRY_URL=$(oc get routes -n default | grep docker-registry | awk '{print $2}')

# Login to the openshift docker registry using 
$ docker login $REGISTRY_URL -u $OC_USER -p $TOKEN

# Tag and push your image to OpenShift's registry
$ docker tag rapid-proxy $REGISTRY_URL/$PROJECT_ID/rapid-proxy
$ docker push $REGISTRY_URL/$PROJECT_ID/rapid-proxy
```

在 OpenShift UI 中，我们现在应该可以看到我们的两个图像流:云应用程序和我们的快速代理图像:
[![](img/1e17f8f2be99bea001b7e56eded2227a.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Image-Streams.png)
部署我们创建的快速代理映像，方法是单击 * 添加到项目= >部署映像 * 并选择显示的值:
[![](img/21b56612c84350133219d943ea8403d3.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Deploy-Proxy.png)
接下来，我们需要输入*RHMAP _ HOST*和*RHMAP _ PORT*环境变量。端口将是 8080 或 8001，但是我们需要找到主机名。该主机名由[open shift SDN](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/sdn.html)使用，并且在对应于我们的开发环境和该项目应用的项目中可用:
[![](img/26556ef9fc1edd6fa8e0e9e178bf7487.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Deploy-Proxy-Variables.png)
例如，我在开发环境中创建和部署了我的应用程序，因此我在 OpenShift UI 中导航到相应的项目，然后从侧菜单中打开 * 应用程序= >服务 * 来查看我从 Red Hat Mobile 创建的服务。很容易发现服务，因为名称的格式是“nodejs-cloudappABCD”，其中“ABCD”是云应用程序 ID 的最后 4 个字符。
最后，导航到 * 应用= >路线 * ，选择我们云应用对应的路线。选择右上角的*Actions =>Edit*，修改如下:
1.  将 * 服务 * 字段改为 * 快速代理。*
2.  将 * 目标端口 * 改为 *  8443。*
3.  确保 * 安全路线 * 被勾选。
4.  设置 *  TLS 终止 * 为 * 穿越。*
5.  设置 * 不安全流量 * 至 * 无。*
[![](img/6b27482f644124bb9dbdae874b7e33e0.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Router-Entry.png)
我们在这里所做的是修改我们的云应用程序的公共路由，这样它现在可以通过阿帕奇 HTTPD 实例路由流量，还可以将 HTTPS 会话管理委托给它。
完成后，我们可以通过打开云应用的 URL 来验证我们的流量是否通过了阿帕奇 HTTPD 服务。结果应该是我们之前在本地测试映像时看到的坏 SSL 证书错误:
[![](img/385e278fa13211b078235bad8c778e4f.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Bad-Request.png)
通过使用我们最喜欢的 HTTP 客户端向 it 部门发送请求，我们可以验证我们的登录是不受限制的。例如，我验证了我能够像这样使用 cURL 登录:

```
$ URL=https://nodejs-cloudappdevezqll-rhmap-rhmap-development.127.0.0.1.nip.io/auth/login
 $ JSON='{"username":"eshortis@redhat.com","password":"redhat2018"}'
 $ curl -k -X POST -H "content-type: application/json" --data $JSON $URL
{"requestId":"1d228803-5f3e-47a2-b496-0f66fb7dde38"}
```

这表明我们的`/auth/login '端点是可访问的，并且从 RapID 服务中检索 RequestID，该 request id 可以返回到客户端应用程序，以便从 RapID 服务中检索 SSL 证书。
客户端应用配置
interde 和 Red Hat Mobile 都为 iOS 和 Android 的本地和混合环境开发提供 SDK。在本节中，我将演示如何使用这些 Cordova SDKs 来保护混合应用程序。
本地克隆项目，并使用以下命令安装项目依赖项:

```
$ npm i -g cordova@6
$ git clone $GIT_URL client-hybrid
$ cd client-hybrid
$ npm install 
```

我们需要在我们的项目中添加一个调解插件，所以现在让我们使用下面的命令来完成它；在发出这个命令之前，您需要从 Intercede 下载 RapID Cordova SDK。下面是我用过的值:

```
$ cordova plugin add $PATH_TO_SDK --variable RAPID_ACCESS_GROUP_IDENTIFIER="com.eshortis.rapiddemo" --variable RAPID_WHITE_LIST="https://nodejs-cloudappdevezqll-rhmap-rhmap-development.127.0.0.1.nip.io"
```

上面提供的值是占位符，您应该用自己的值替换。下面简单解释一下每个变量:
*   RAPID _ ACCESS _ GROUP _ IDENTIFIER——用于在 iOS 钥匙串中存储凭据的标识符。
*   RAPID_WHITE_LIST -我们将通过双向 TLS 保护的 URL 列表。
最后，我们需要添加一个登录组件和一些 JavaScript 来实现登录，并使用 RapID SDK 获取客户端证书。我创建了两个包含代码的 GitHub Gists】
*   [index.html](https://gist.github.com/evanshortiss/379d7a8f7ad72704a069bf55510ca749)
*   [hello . js](https://gist.github.com/evanshortiss/53ff15fd23d78f1f43d10fbee66259c0)
一旦我们将这两个文件的内容复制到本地存储库中各自的文件中，在 Xcode 中打开该项目，并验证我们的应用程序的 * 功能 * 下的 * 钥匙串共享 * 选项已启用，并且正确的访问组已被填充，如下所示:
[![](img/1fffbb61d5ccfc9ddcb392184e3e29c8.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Xcode-Capabilities.png)
按下运行按钮或使用⌘+R 快捷键启动应用程序。一旦应用程序启动，我们需要使用 iPhone 的 Home 键来关闭它，这有点不直观。一旦应用程序关闭，点击并拖动我们使用 OpenSSL 生成的*server . CRT*文件到 iPhone 上；在生产应用程序中，这是不必要的，因为我们将使用由可信的证书颁发机构签名的证书。
[![](img/2093ef6d9ad25582f18346469c5514f4.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Install-Certificate.png)
接下来，我们需要导航到设置应用程序，选择*General =>About =>Certificate Trust Settings*，并为我们刚刚安装的证书启用 * 完全信任根证书*:
[![](img/fad9b39b2fff9cc5fe39faae17531758.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Install-Certificate-Root.png)
现在，重启应用程序并输入用户名和密码。来自红帽移动 SDK 的 * 云 * 函数用于将我们的凭证发送到后端*/auth/log in*端点。成功登录后，应用程序将使用登录响应返回的“requestId”向 RapID 请求匹配的 SSL 证书。使用 RapID SDK 获取证书后，我们输入一个 PIN 来保护它:
[![](img/4543cdbfb36779fe208132e7e2575dc1.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercded-RapID-PIN-Entry.png)
在我们的 PIN 和 SSL 证书被存储在设备钥匙串中之后，我们准备好访问我们的*/API/hello*端点。在显示的字段中输入名称，然后按“从云端问好”按钮。这将使用我们的参数调用 RapID SDK 的*sendRequestEx*函数，执行使用双向 TLS 保护的 HTTPS 请求，并从运行在 Red Hat 移动应用程序平台上的 Node.js 应用程序获取我们的“Hello World”响应:
[![](img/99849d2dae66431dfff2b5223441cf91.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/03/Intercede-Successful-Request.png)
结论
我们已经成功地将 Intercede 的快速解决方案集成到我们的应用程序中，并实现了以下目标:
1.  一个双向 TLS 来阻止未知设备对我们的 JSON API 的请求。
2.  在设备钥匙串中嵌入了证书，我们开发的任何应用程序都可以共享该证书。
3.  能够通过使用 RapID certificate 和 SDK 在其他应用程序中对用户进行身份验证来改善用户体验。

																*Last updated:
							April 2, 2018*

```
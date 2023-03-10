# 如何使用 Red Hat 单点登录、熔丝和 3scale 保护微服务

> 原文：<https://developers.redhat.com/blog/2019/10/30/how-to-secure-microservices-with-red-hat-single-sign-on-fuse-and-3scale>

在本文中，我们将通过使用 OpenID Connect 等协议，在 [Red Hat 单点登录](https://developers.redhat.com/blog/2019/02/06/using-a-public-certificate-with-red-hat-single-sign-on-keycloak/)和 [3scale](https://developers.redhat.com/products/3scale/overview) 的支持下，介绍微服务安全概念。在使用基于[微服务](https://developers.redhat.com/topics/microservices/)的架构时，必须仔细设计分布式的、深入的用户身份和访问控制。在这里，我们将从现实的角度一步一步地详细介绍这些工具的集成。

本文举例说明了可以安全运行您的业务的工具的使用，避免使用自制的解决方案，并通过使用 API 网关保护您的服务，防止您的应用程序暴露于公共网络。API 网关的使用还提供了额外的访问控制、货币化和分析。

![security](img/f68f28933a73d992f19b9f06ad09aae7.png)

| **技术** | **版本** |
| --- | --- |
| Spring Boot | 2.1.8 .释放 |
| [阿帕奇骆驼](https://camel.apache.org/) | [7 . 4 . 0 . fuse-740036-red hat-00002](https://www.redhat.com/en/technologies/jboss-middleware/fuse)
(w/spring boot 1 . 5 . 22 . release) |
| [3 刻度](https://www.3scale.net/) | [2.6](https://access.redhat.com/containers/#/product/RedHat3scaleApiManagement) |
| [【红帽单点登录】](https://access.redhat.com/products/red-hat-single-sign-on) | [7 . 3 . 3](https://access.redhat.com/containers/#/product/RedHatSingleSign-on)
(基于[key cloak 4.8](https://access.redhat.com/articles/2342881)) |

**TL；DR:** 这是一个如何用 Red Hat 单点登录(Keycloak)和 3scale 保护 API 的演示。

![](img/0f73d655582f3de947fec3e737da4e17.png)

这是一篇很长的文章，有一步一步的说明、产品截图和架构概念。所有源代码都托管在 GitHub 上。

**注:**这是概念证明。在生产环境中，将需要关于可伸缩性、安全性(细化)和使用适当的 CA 信任证书的附加配置。

## 用例场景

本教程的主要目的是通过一个完整的用例场景来实现关于微服务安全性的概念。提供了一个 web 应用程序来促进对所使用的所有 API 调用和授权的更自然的理解。

所有 API 目录如下所示。

#### `auth-integration-api`终点

**:8081**

| **法** | **URI** | **描述** |
| --- | --- | --- |
| 获取 | /健康 | API 执行器嵌入式健康。 |
| 获取 | /指标 | API 执行器嵌入式指标。 |

**:8080**

| **法** | **URI** | **描述** | **担保？** |
| --- | --- | --- | --- |
| 发帖 | /API/v1/产品 | 创造新产品。 | 真 |
| 删除 | /api/v1/product/* | 按 ID 删除产品。 | 真 |
| 放 | /api/v1/product/* | 按 ID 更新产品。 | 真 |
| 获取 | /API/v1/产品$ | 检索所有产品 | 真 |
| 获取 | /api/v1/product/* | 同上。 | 真 |
| 获取 | /API/v1/状态 | 检查集成 API 健康状况。 | 真 |
| 获取 | /API/v1/产品/状态 | 检查产品 API 健康状况。 | 真 |
| 获取 | /API/v1/供应商/状态 | 检查供应商的 API 健康状况。 | 真 |
| 获取 | /API/v1/库存/状态 | 检查库存 API 状况。 | 真 |
| 获取 | /API/v1/库存/维护 | 调用库存 API 维护。 | 真 |
| 获取 | /API/v1/供应商/维护 | 调用供应商 API 维护。 | 真 |

#### `stock-api`终点

| **法** | **URI** | **描述** |
| --- | --- | --- |
| 获取 | /api/v1/sync | 库存维护 |
| 获取 | /执行器/健康 | 供应商维护 |

#### `supplier-api`终点

| **法** | **URI** | **描述** | **担保？** |
| --- | --- | --- | --- |
| 获取 | /api/v1/sync | 供应商维护 | 真 |
| 获取 | /执行器/健康 | 供应商维护 | 真 |

#### `product-api`终点

| **法** | **URI** | **描述** | **担保？** |
| --- | --- | --- | --- |
| 获取 | /API/v1/产品 | 检索所有产品。 | 真 |
| 获取 | /api/v1/product/{id} | 通过 ID 检索产品。 | 真 |
| 发帖 | /API/v1/产品 | 创造新产品。 | 真 |
| 放 | /api/v1/product/{id} | 按 ID 更新产品。 | 真 |
| 删除 | /api/v1/product/{id} | 按 ID 删除产品。 | 真 |

## 安全实验室

每个端点都有自己的特殊性，所以为了推动我们的测试场景，我以三个简单的问题结束:

*   这个 API 会受到集成层(FUSE)的保护吗？
*   这个 API 会在 3scale AMP 上作为一个独特的服务公开吗？(此因素为外部客户启用 API 自助订阅。)
*   这个 API 将由 RHSSO (Keycloak)管理吗？它有自己的客户端 id、组和角色吗？

所以，我想出了下面的需求矩阵:

![](img/4a2b2da0ab4c839f54d8aafafe3fbc8b.png)

正如我们所看到的，每个 API 都有差异，我们将努力在这个微服务安全实验室中演示每个 API。

### 步骤 1:项目创建

按如下方式创建此项目:

| **导出**PROJECT _ NAMESPACE =微服务
#登录 openshift 平台
**oc 登录** https://master。< >。com:443-token =<>
#新建项目
**oc 新建-项目** 微服务-安全-描述= "微服务安全"-显示-名称= "微服务-安全" |

### 步骤 2:部署 Nexus 原型

在`microservices`名称空间中提供一个 Sonatype Nexus 实例。详细说明可以在这个[自述](https://github.com/aelkz/microservices-security/blob/master/README-NEXUS.md)中找到。

### 步骤 3:3 规模放大器部署

您还必须在您的 Red Hat Openshift 容器平台中提供一个 3 倍的放大器。关于如何安装 3scale 应用程序，请参考[文档](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.6)。

### 步骤 4: Red Hat 单点登录部署

此外，为这个示例安装 Red Hat 单点登录。参考[文档](https://access.redhat.com/products/red-hat-single-sign-on)了解如何安装 RHSSO 应用程序。

### 步骤 5: Nexus 环境设置

按如下方式设置您的 Nexus 环境:

| **导出**PROJECT _ NAMESPACE =微服务-安全
**git 克隆**[https://github.com/aelkz/microservices-security.git](https://github.com/aelkz/microservices-security.git)cd 微服务-security/
#下载 maven settings.xml 文件
**curl**-o MAVEN-settings-template . XML-s[https://raw . githubusercontent . com/aelkz/microservices-security/master/_ configuration/NEXUS/MAVEN-settings-template . XML](https://raw.githubusercontent.com/aelkz/microservices-security/master/_configuration/nexus/maven-settings-template.xml)
#使用您的 nexus openshift 路线更改镜像 URL【T11 -template = ' { { . spec . host } } ')/repository/MAVEN-releases/
**export**MAVEN _ URL _ snapshot s = http://$(oc get route NEXUS 3-n $ { NEXUS _ NAMESPACE }-template = ' { { . spec . host } } ')/repository/MAVEN-snapshot s/
awk-v path = " $ MAVEN _ URL " '/<URL< * < /，">" path "<")1 ' maven-settings-template . XML>maven-settings . XML
RM-fr maven-settings-template . XML |

### 步骤 6:创建 Red Hat 容器目录机密

为了导入 Red Hat 容器图像，您必须在 OpenShift 上创建一个密码并设置您的凭证:

| 注。为了导入 Red Hat 容器图像，您必须在 openshift 上设置您的凭证。参见:[https://access.redhat.com/articles/3399531](https://access.redhat.com/articles/3399531)
# config . JSON 可在:/var/lib/origin/找到。openshift 主节点上的 docker/
#用你的容器凭证创建一个秘密**导出**$ PROJECT _ NAMESPACE =微服务-安全**oc 删除秘密**red hat . io-n $ PROJECT _ NAMESPACE
**oc 创建秘密** 泛型" redhat.io" - from-file=。docker config JSON = config . JSON-type = kubernetes . io/docker config JSON-n $ PROJECT _ NAMESPACE
**oc create secret**generic red hat . io-from-file =。docker config JSON = config . JSON-type = kubernetes . io/docker config JSON-n $ PROJECT _ NAMESPACE
**oc secrets link**default red hat . io-for = pull-n $ PROJECT _ NAMESPACE
**oc secrets link**builder red hat . io-n $ PROJECT _ NAMESPACE |

### 步骤 7: RHSSO 领域配置

在这一步中，我们将在 RHSSO 上配置`realms`来注册所有五个应用程序。

1.  正在登录 RHSSO。
2.  使用默认设置创建三个领域:
    *   `3scale-api`
    *   `3scale-admin`
    *   `3scale-devportal`

创建完领域后，您将得到如下内容:

![](img/ff7ab2d1355a0123db5e3dea07694ff7.png)

4.  在 **3scale-api** 领域中，使用以下定义创建客户端`3scale`:

![](img/b962135cd327de828b22dec0e0a3f44e.png)

将这些字段留空:

5.  在**服务帐户角色**选项卡上，从`realm-management`分配角色`manage-clients`。
6.  复制并保存为该客户端生成的`client-secret`。这个秘密稍后将用于在 3scale 上配置 OAuth 服务认证，并且看起来像这样:`823b6ek5-1936-42e6-1135-d48rt3a1f632`。
7.  在领域`3scale-api`下，使用以下定义创建一个新用户:

![](img/44a35c05d793e63f0c139db131821bea.png)

8.  使用`temporary=false`在`Credentials`选项卡上为该用户设置新密码。
9.  在`Details`选项卡上将`Email Verified`属性设置为`true`。

### 步骤 8:3 扩展微服务配置

在这一步中，我们注册 API 并对它们进行配置，以启用与 RHSSO 的 3scale 自动同步。让我们设置`auth-integration-api`和`supplier-api`:

1.  在 3scale 管理门户上创建新的 API。您可以点击主仪表板上的**新 API** 按钮:

![](img/36153dab93e88b03f9cd5685f3188129.png)

这个新的 API 将代表我们之前部署的`auth-integration-api`:

![](img/8682146504444fee7c84de73dafb4482.png)

2.  浏览**集成**下的**配置**菜单，设置 API 映射和安全性:

![](img/f10936a6d44b18443a46f62531d54fca.png)

3.  为网关选择 **APIcast** :

[![](img/7544005a743436f704c5d36d95420d09.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/14.png)

4.  在**集成设置**中选择 **OpenID Connect** :

[![](img/9276051264b675797939c08351c5f959.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/15.png)

**注意:**选择 OpenID Connect 是因为我们将使用 RHSSO 提供的 OAuth2 功能来保护我们的 API。

5.  点击:

[![](img/96b8396c7bbd1662444029892bff4427.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/16.png)

6.  定义**私有库 URL** (您的`auth-integration-api` URL)，暂存公共库 URL ，以及**生产公共库 URL** :

![](img/e88c26d57982aabdcebd75007d7f4efc.png)

**注意:**在每个 URL 下设置正确的域，这将成为你在 OpenShift 上的 API 路由。

7.  根据下表定义此 API 的所有映射规则:

| **动词** | **图案** | **+** | **公制或法** |
| --- | --- | --- | --- |
| 发帖 | /API/v1/产品 | one | 打击 |
| 删除 | /api/v1/product/* | one | 打击 |
| 放 | /api/v1/product/* | one | 打击 |
| 获取 | /API/v1/产品$ | one | 打击 |
| 获取 | /api/v1/product/* | one | 打击 |
| 获取 | /API/v1/状态 | one | 打击 |
| 获取 | /API/v1/产品/状态 | one | 打击 |
| 获取 | /API/v1/供应商/状态 | one | 打击 |
| 获取 | /API/v1/库存/状态 | one | 打击 |
| 获取 | /API/v1/库存/维护 | one | 打击 |
| 获取 | /API/v1/供应商/维护 | one | 打击 |

8.  定义此 API 的验证机制:

![](img/ebf405b4035d099dfcb925082b3cb95f.png)

9.  配置所需的 API 策略，以便在 OpenShift 容器平台内部的资源之间实现正确的通信:

![](img/69b805dd809288c6f9524c3dab7aa554.png)
Please follow the next steps carefully:

1.  在 **OIDC 授权流程**部分选择**授权码流程**、**服务账户流程**和**直接访问授权流程**。
2.  在**凭证位置**设置**为 HTTP 头**。
3.  在**策略**部分，添加(按此顺序)`CORS`和`3scale APIcast`。
4.  展开 CORS 配置，并设置以下内容:
    *   `ALLOW_HEADERS`为每个输入数组添加一个 1x1:

```
Enabled=checked
ALLOW_HEADERS
```

| **3 缩放 CORS 策略:标题** |
| --- |
| 内容类型 |
| 授权 |
| 内容长度 |
| X-被请求-与 |
| 原点 |
| 接受 |
| X-被请求-与 |
| 内容类型 |
| 访问控制请求方法 |
| 访问控制请求报头 |
| 接受编码 |
| 接受语言 |
| 连接 |
| 主持人 |
| 推荐人 |
| 用户代理 |
| 访问控制允许来源 |
| X-Business-FooBar |

**注意:**最后一个标题仅用于测试目的。

```
allow_credentials=checked
ALLOW_METHODS 
```

| **3 缩放 CORS 策略:HTTP 方法** |
| --- |
| 获取 |
| 头 |
| 发帖 |
| 放 |
| 删除 |
| 选项 |

5.  将`allow_origin`留空，其余为默认值。
6.  保存 CORS 配置。

**注意:**每次更改后，记得通过点击
:![](img/3ec8948c8629c5dd670da4e8961cec00.png)将暂存配置升级到生产

您的`auth-integration-api`已经可以使用了。

对供应商 API 重复本节中相同的步骤。这个 API 只有两个映射规则:

| **动词** | **图案** | **+** | **公制或法** |
| --- | --- | --- | --- |
| 发帖 | /api/v1/sync | 1 | 点击量 |
| 获取 | /执行器/健康 | 1 | 点击量 |

### 步骤 9:3 扩展微服务应用计划

让我们定义 API 的应用程序计划。这些计划将在客户注册时用于创建新的应用程序:

![](img/35b235cd276a99d1f01d7f8cc9c42314.png)

点击**应用/应用计划**菜单下的 [![](img/e8b695b5656ee6e073bc20db23154079.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/23.png) ，然后设置如下配置:

![](img/79293023ca6ea8afa1720b7dae43618e.png)

完成后，点击**发布**链接发布申请计划。

遵循`Supplier API`的相同步骤。记得也发布这个应用程序计划。

完成前面的所有步骤后，您将得到如下结果:

![](img/596292b204299999a079cdf69f74ebb3.png)

### 步骤 10:3 大规模微服务应用

浏览**受众**菜单，在**账户/列表**下，点击 [![](img/46da56ef512135baf0e7bb7ae3a1a6b7.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/25.png) 创建新账户。然后，使用您的凭据创建一个新帐户，用于本演示:

![](img/d0b19d62f309e2993fbbce3fd357fe9c.png)

此操作将创建一个新的 3scale 应用程序。如果无法创建应用程序，只需点击 [![](img/589b1b9e342e4a56d56244c36e5353b3.png)](https://raw.githubusercontent.com/aelkz/microservices-security/master/_images/27.png) 链接。

这个新应用程序是为使用`auth-integration-api`而创建的。客户端 id 和客户端密码将自动生成，并由`zynnc-que`3 规模应用程序推送到`3Scale-api`领域中的 RHSSO:

![](img/88ec05543426d41909984a80eddcf212.png)

在 3scale 上创建 API 定义后，检查生成的客户机是否被推入 RHSSO 上的`3scale-api`领域。如果您使用自签名证书，您需要进行额外的配置，以便启用`zynnc-que`3 规模应用程序同步。请参考[文档:SSL 问题故障排除](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.6/html-single/operating_3scale/index#troubleshooting_ssl_issues)和[配置 Zync 使用自定义 CA 证书](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.4/html-single/api_authentication/index#zync-oidc-integration)。

要解决此问题，请继续安装自签名证书:

| **导出**three scale _ NAMESPACE = 3 scale 26
**导出**three scale _ ZYNC _ QUE _ POD = $(oc get pods-selector deployment config = ZYNC-que-n 3 scale 26 &#124; { read line 1；读第 2 行；回显" $ line 2 "；} &#124; awk“{ print $ 1；}')
**导出**RHSSO _ URI = SSO 73 . apps .<YOUR-DOMAIN>。com
echo &#124; OpenSSL s _ client-showcerts-servername $ { RHSSO _ URI }-connect $ { RHSSO _ URI }:443 2>/dev/null &#124; sed-ne '/-BEGIN CERTIFICATE-/，/-END CERTIFICATE-/p '>self-signed-cert . PEM

#先验证连接！必须返回 HTTP/1.1 200 OK
**curl**-v https://$ { RHSSO _ URI }/auth/realms/master-cacert 自签名-cert . PEM
**oc exec**$ { three scale _ ZYNC _ QUE _ POD } cat/etc/PKI/TLS/cert . PEM>ZYNC-1# oc 删除配置图 zync-que-ca-bundle
**oc 创建配置图**zync-que-ca-bundle-from-file =。/zync-que . PEM-n $ { three scale _ NAMESPACE }
**oc label config map**zync-que-ca-bundle app = 3 scale-API-management-n $ { three scale _ NAMESPACE }
**oc set volume**DC/zync-que-overwrite-add-name = zync-que-ca-bundle-mount-path/etc/PKI/TLC**【oc 补丁】**【DC/znc-que-type = JSON-p '[{ " op ":" add "，" path ":/spec/template/spec/containers/0/volumes mounts/0/subpath "，" value ":" znc-que . PEM " }]'-n $ { threw _ namespace }
#等待容器重新启动，并检查日志中是否有任何问题。
**oc 日志**-f po/$ { three scale _ ZYNC _ QUE _ POD }# **瞧！**您使用自签名证书将 3Scale 与 RHSSO 同步。 |

### 步骤 11: Node.js web 应用程序部署

在这一步，我们将使用基于 Angular 和 Bootstrap 的合适的 NodeJS webapp 测试所有场景。该应用程序旨在简化理解过程，并可用于使用我们的`Jon Doe`用户帐户阐明授权行为:

| # Deploy nodejs-web 应用
#【https://access.redhat.com/containers/?】tab = images #/registry . access . red hat . com/RHS cl/nodejs-10-rhel 7**oc import-image**RH SCL/nodejs-10-rhel 7-from = registry . red hat . io/RH SCL/nodejs-10-rhel 7-n open shift-confirm**导出**APIS _ NAMESPACE =微服务-安全
**导出**three scale _ NAMESPACE = 3 scale 26
**导出**RHSSO _ NAMESPACE = SSO 73
**导出**RHSSO _ URL = https://$(oc get route-n $)} ')/auth
**export**three scale _ APP _ DOMAIN =<YOUR-DOMAIN>。com
**export**three scale _ API _ URL = https://$(oc get routes-n $ { three scale _ NAMESPACE } &#124; grep auth-integration &#124; grep production &#124; awk ' { print $ 2；}')
**导出**INTEGRATION _ HEALTH _ URL = http://$(oc get routes-n $ { APIS _ NAMESPACE } &#124; grep auth-INTEGRATION &#124; grep metrics &#124; awk ' { print $ 2；}')echo-e \
"AUTH _ CLIENT _ ID=<AUTH _ INTEGRATION _ CLIENT _ ID>\ n "
"AUTH _ URL= $ { RHSSO _ URL } \ n "
"AUTH _ REALM= 3 scale-API \ n "
""
"AUTH _ CLIENT _ SECRET= 1 D1 beebcf CJD 002d 51 be 7a 346 ab 987 p \ n "
"NODE _ TLS _ REJECT _ UNAUTHORIZED= 0 \ n "
>temp**sed**“s/^.//g " temp>>nodejs-config . propertiesrm -fr 温度# oc 删除 config map nodejs-we b-config
**oc 创建 config map**nodejs-we b-config \
-from-literal = AUTH _ CLIENT _ ID = \
-from-literal = AUTH _ URL = \
-from-literal = AUTH _ REALM = \
-from-literal = key cloak# oc delete all-lapp = nodejs-web
oc new-appnodejs-10-rhel 7:latest ~[https://github.com/aelkz/microservices-security.git](https://github.com/aelkz/microservices-security.git)-name = nodejs-we B- context-dir =/web app-n $ { APIS _ NAMESPACE }#定义好属性后，在 nodejs-web 容器上设置环境变量。
**oc set env**-from = config map/nodejs-we b-config DC/nodejs-we b-n $ { APIS _ NAMESPACE } |

**注意:**在`nodejs-web`容器上设置所有的环境变量，以便正确地启用 API 调用。

公开 web 应用程序路由:

| **oc 创建路由边缘**-service = nodejs-we B- cert = web app/server . cert-key = web app/server . key-n $ { APIS _ NAMESPACE } |

### 步骤 12:应用程序设置和角色

现在我们创建我们的应用程序`roles`。这些角色被分配给将用于登录我们的 web 应用程序的应用程序用户。

访问代表先前由 3scale `application`进程注册的`auth-integration`客户端的 client-id，然后转到客户端的**设置**选项卡并应用附加配置。有效重定向 URIs 包括:

```
http://*
https://*
```

有效的网络来源很简单:

```
*
```

转到 RHSSO (Keycloak)上**客户端**菜单的**角色**选项卡，创建以下角色:

![](img/cf5d5ab3a9e00963590a5bcbdcb1308c.png)

对`Supplier API`客户端重复相同的步骤。该客户端将只定义一个角色:

![](img/2a2f7327e379de3410c2edf9f44bd8b0.png)

**注意:**这个客户端也是通过 3scale 生成的。(您必须创建两个应用程序:一个用于`auth-integration-api`，另一个用于`supplier-api`。

### 步骤 13:用户角色

在这一步，我们将所有的`client`角色分配给`john doe`用户，`service-account`用户将处理`auth-integration-api`内部的`supplier-service`调用。

在**用户**菜单中的 **John Doe** 用户详细信息页面上，转到**角色映射**选项卡。按照下图将所有角色分配给用户:

![](img/50195555eb823466dcbf380a0ae1ae9f.png)

在第 7 步中，我们创建了`John Doe`用户。我们需要创建另一个用户，它将被用作`service-account`来调用`auth-integration-api`中的`Supplier API`(参见 [application.yaml](https://raw.githubusercontent.com/aelkz/microservices-security/master/product/src/main/resources/application.yaml) 的第 123 行)。这个用户也有一个密码，所以用`12345`重置它的凭证。这个用户的名字可以是 3scale 生成的`Supplier API` client-id 的`id`加上`_svcacc`后缀(见 [application.yaml](https://raw.githubusercontent.com/aelkz/microservices-security/master/product/src/main/resources/application.yaml) 的第 131 行)。

我们还需要为这个用户分配`SUPPLIER_MAINTAINER`角色。

**注意:**这个过程是作为`token-exchange`机制的替代，但是我们可以通过使用`token-exchange`特性对使用第三方 API 的其他可能性进行更详细的研究。

最后，创建一个`realm-admin`用户。这个用户将使用 RHSSO REST API。分配凭证`12345`和所有`realm-management`角色:

![](img/ab49a80e2dc673a53c53c698985a1430.png)

最终，`3scale-api`领域中会有三个用户:

![](img/e57d675aeba8256f78a74700e91fc69f.png)

### 步骤 14:在 Nexus 上归档 SSO-common 库 jar

为了确保`auth-integration-api` (Fuse)正常工作，我们需要归档一个库，然后使用该库在 Red Hat 单点登录之上启用身份验证和授权:

| #注意:为了确保 auth-integration-api (Fuse)正常工作，我们需要归档一个库，该库将用于在 Red Hat Single Sign-On (Keycloak)之上提供身份验证和授权功能。然后，将在 auth-integration-api 上使用这个库来启用这样的功能。#在 nexus 上部署 auth-sso-common 库
**export**NEXUS _ NAMESPACE = cicd-dev tools
**export**MAVEN _ URL = http://$(oc get route NEXUS 3-n $ { NEXUS _ NAMESPACE }-template = ' { { . spec . host } } ')/repository/MAVEN-group/
**mvn 清理包部署**-dnexusreleaserepour = $ MAVEN _ URL _ RELEASES-DnexusSnapshotRepoUrl = $ MAVEN _ URL _ snapshot s-s ./MAVEN-settings . XML-e-X-pl auth-SSO-common |

这个动作在 Nexus 上创建了以下工件:
![](img/3a6050edccfde98ded154da0a99b545d.png)

### 步骤 15:微服务部署

检索 RHSSO 领域公钥:

| **导出**RHSSO _ REALM = 3 scale-API
**导出**RHSSO _ URI = SSO 73 . apps .<YOUR-DOMAIN>。com
**export**TOKEN _ URL = https://$ { RHSSO _ URI }/auth/REALM/$ { RHSSO _ REALM }/protocol/OpenID-connect/TOKEN
**export**three scale _ REALM _ USERNAME = admin
**export**three scale _ REALM _ PASSWORD = three**= $(curl-k-X POST " $ TOKEN _ URL " \** **-H " Content-Type:application/X-www-form-urlencoded " \
-d " USERNAME = $ three scale _ REALM _ USERNAME " \
-d " PASSWORD = $ three scale _ REALM _ PASSWORD " \
-d " grant _ Type = PASSWORD " \* access _ token ":"//g“&#124; sed s/”。*//g')** ****导出**REALM _ KEYS _ URL = https://$ { RHSSO _ URI }/auth/admin/realms/$ { RHSSO _ REALM }/KEYS

**RSA _ PUB _ KEY**= $(curl-k-X GET " $ REALM _ KEYS _ URL " \
-H "授权:无记名$TKN" \
&#124; jq -r '。键[] &#124;选择(。type=="RSA") &#124;。public key’)

#创建一个有效。pem 证书
REALM _ CERT = $ RHSSO _ REALM . PEM
echo**"-BEGIN CERTIFICATE-"**>【REALM _ CERT；echo $ RSA _ PUB _ KEY>>$ REALM _ CERT；回显 **" -结束证书-"**>>$ REALM _ CERT

#检查生成的。pem 证书
# fold-s-w 64 $ REALM _ CERT>$ RHSSO _ REALM . fixed . PEM
# OpenSSL x509-in $ RHSSO _ REALM . fixed . PEM-text-noout
# OpenSSL x509-in $ RHSSO _ REALM . fixed . PEM-noout-issuer-fingerprint**  |

然后，部署`parent`项目:

| #在 nexus 上部署父项目
**mvn clean package Deploy**-dnexusreleaserepour = $ MAVEN _ URL _ RELEASES-DnexusSnapshotRepoUrl = $ MAVEN _ URL _ snapshot-s ./MAVEN-settings . XML-e-X-N |

现在，展开`stock-api`:

| # oc delete all-lapp = stock-API
oc new-appopen JDK-8-rhel 8:latest ~[https://github.com/aelkz/microservices-security.git](https://github.com/aelkz/microservices-security.git)-name = stock-API-context-dir =/stock-build-env = ' MAVEN _ MIRROR _ URL = ' $ { MAVEN _ URL }-e MAVEN _ MIRROR _ URL = $ { MAVEN _ URL }oc 补丁 SVCstock-API-p ' { " spec ":{ " ports ":[{ " name ":" http "，" port":8080，" protocol":"TCP "，" target port ":8080 }]} 'oc label SVCstock-API monitor = spring boot 2-API |

使用提供的`configmap`和`secret`设置所需的变量:

| **oc 创建** -f 配置/config map/stock-API-env . yml-n $ { PROJECT _ NAMESPACE }
**oc 创建** -f 配置/secret/stock-API . yml-n $ { PROJECT _ NAMESPACE }**导出**APP = stock-API
**oc set env**DC/$ { APP }-from = secret/stock-API-secret**oc set env**DC/$ { APP }-from = config map/stock-API-config |

部署`supplier-api` **、**，但是在继续之前首先检查`application.yaml`文件中的所有设置。此处的属性必须更新以反映您的实际环境:

*   `rest.security.issuer-uri`第 61 行。
*   `security.oauth2.resource.id`第 71 行。
*   `security.oauth2.resource.jwt.key-value`第 75 行。

| # oc delete all-lapp = supplier-API
**oc new-app**open JDK-8-rhel 8:latest ~[https://github.com/aelkz/microservices-security.git](https://github.com/aelkz/microservices-security.git)-name = supplier-API-context-dir =/supplier-build-env = ' MAVEN _ MIRROR _ URL = ' $ { MAVEN _ URL }-e MAVEN _ MIRROR _ URL = $ { MAVEN _ URL }**oc 补丁 SVC**supplier-API-p ' { " spec ":{ " ports ":[{ " name ":" http "，" port":8080，" protocol":"TCP "，" target port ":8080 }]} '**oc label svc** 供应商-api monitor=springboot2-api |

使用提供的`configmap`和`secret`设置所需的变量:

| **oc 创建** -f 配置/config map/supplier-API-env . yml-n $ { PROJECT _ NAMESPACE }
**oc 创建** -f 配置/secret/supplier-API . yml-n $ { PROJECT _ NAMESPACE }**导出**APP = supplier-API

**oc set env**DC/$ { APP }-from = secret/supplier-API-secret
**oc set env**DC/$ { APP }-from = config map/supplier-API-config |

部署`product-api`，但是在继续之前，再次检查`application.yaml`文件中的所有设置。以下属性必须更新以反映您的实际环境:第 61 行的`rest.security.issuer-uri`和第 75 行的`security.oauth2.resource.jwt.key-value`:

| # oc delete all-lapp = product-API
**oc new-app**open JDK-8-rhel 8:latest ~[https://github.com/aelkz/microservices-security.git](https://github.com/aelkz/microservices-security.git)-name = product-API-context-dir =/product-build-env = ' MAVEN _ MIRROR _ URL = ' $ { MAVEN _ URL }-e MAVEN _ MIRROR _ URL = $ { MAVEN _ URL }**oc 补丁 svc** 产品-API-p ' { " spec ":{ " ports ":[{ " name ":" http "，" port":8080，" protocol":"TCP "，" target port ":8080 }]} '**oc label svc** 产品-api monitor=springboot2-api |

使用提供的`configmap`和`secret`设置所需的变量:

| **oc 创建** -f 配置/config map/product-API-env . yml-n $ { PROJECT _ NAMESPACE }
**oc 创建** -f 配置/secret/product-API . yml-n $ { PROJECT _ NAMESPACE }**导出**APP = supplier-API

**oc set env**DC/$ { APP }-from = secret/product-API-secret
**oc set env**DC/$ { APP }-from = config map/product-API-config |

### 步骤 16:集成部署(FUSE)

既然已经部署了微服务 API，让我们来部署集成层:

| #导入新的 spring-boot camel 模板
**curl**-o s2i-micro services-fuse 74-spring-boot-camel . YAML-s[https://raw . githubusercontent . com/aelkz/micro services-security/master/_ configuration/open shift/s2i-micro services-fuse 74-spring-boot-camel . YAML](https://raw.githubusercontent.com/aelkz/microservices-security/master/_configuration/openshift/s2i-microservices-fuse74-spring-boot-camel.yaml)**oc 删除模板**s2i-微服务-fuse 74-spring-boot-camel-n $ { PROJECT _ NAMESPACE }
**oc 创建**-n $ { PROJECT _ NAMESPACE }-f s2i-微服务-fuse 74-spring-boot-camel . YAML#注。您可能需要检查..self-signed.yaml 模板，因为它使用自定义的图像流来与自签名证书一起使用。(详见附录-README.md 获取信息)
**导出**NEXUS _ NAMESPACE = cicd-dev tools
**导出**PROJECT _ NAMESPACE =微服务-安全
**导出**APP = auth-integration-API
**导出**先前的模板对服务、路线和组定义进行了一些修改。
# oc delete all-lapp = $ { APP }
**oc new-APP**-TEMPLATE = $ { CUSTOM _ TEMPLATE }-NAME = $ { APP }-build-env = ' MAVEN _ MIRROR _ URL = ' $ { MAVEN _ URL }-e MAVEN _ MIRROR _ URL = $ { MAVEN _ URL }-param GIT _ REPO = $ { APP _ GIT }-param APP _ NAME = $ { APP }-param ARTIFACT _ DIR#如果使用不同的标签，则使用此参数(示例)
#-param BUILDER _ VERSION = 2.0#检查创建的服务:
# 1 用于默认应用上下文，1 用于/度量端点。
**oc get SVC**-n $ { PROJECT _ NAMESPACE } &#124; grep $ { APP _ NAME }#为了 auth-integration-api 调用其他 api，我们需要更改它的配置:
**curl**-o application . YAML-s https://raw . githubusercontent . com/aelkz/microservice-security/master/_ configuration/open shift/auth-integration/application . YAML#注。如果您更改了服务或应用程序的名称，则需要使用您的定义编辑和更改下载的 application.yaml 文件。#创建一个配置映射并为授权集成 api 挂载一个卷
**oc 删除配置映射**$ { APP }-n $ { PROJECT _ NAMESPACE }**oc create**-f configuration/config map/auth-integration-API-env . yml-n $ { PROJECT _ NAMESPACE }
**oc create**-f configuration/secret/auth-integration-API . yml-n $ { PROJECT _ NAMESPACE }**oc set env**DC/$ { APP }-from = secret/auth-integration-API-secret**oc set env**DC/$ { APP }-from = config map/auth-integration-API-config |

**注意:**所有的应用程序角色在源代码上都有前缀`ROLE_`。如果您想在`../configuration/security/JwtAccessTokenCustomizer.java`类的第 80 行修改这个前缀。在 RHSSO 上，这些角色注册时没有这个前缀。参见此[堆栈溢出](https://stackoverflow.com/questions/33205236/spring-security-added-prefix-role-to-all-roles-name)参考。

### 步骤 17:使用 Red Hat 单点登录测试 NodeJS 应用程序

在浏览器中打开 node.js web 应用程序

| **导出** 微服务 _ 名称空间=微服务-安全
**回显**http://$(oc get route nodejs-we b-n $ {微服务 _ 名称空间}-template = ' { { { . spec . host } } ') |

如果您使用的是自签名证书，浏览器将请求授权来打开不安全的 URL。浏览菜单，点击每个按钮查看最终结果，测试所有操作。如果某个操作返回 401 或 403，则可能存在 3 规模的待定配置，或者某个应用程序缺少凭据或凭据无效。如果您得到 HTTP 错误 500，可能是应用程序不可用。尝试更改`Jon Doe`角色，并在刷新访问令牌后检查每种情况:

![](img/8209e9e394e14acfbe3c2915f92bac5b.png)

(点击图片查看大图。)

![](img/76f786ebc53838e47394d4603ea3dc4e.png)

我希望你喜欢这个教程。由于涉及到所有的 OAuth2 适配器和安全机制，故障排除有些困难。如果您想改进某些东西，或者向此概念验证添加更多内容，请告诉我。谢谢你。

### 参考

*   [API 密钥生成器](https://codepen.io/corenominal/pen/rxOmMJ)
*   [JWT 密钥生成器](http://jwt.io)
*   [OpenID 连接调试器](https://openidconnect.net)
*   [从 JSON 或 JSON-Schema 生成普通的旧 Java 对象](http://www.jsonschema2pojo.org)
*   [使用 Keycloak 和 Spring Oauth2 (@bcarun)保护 REST API](https://medium.com/@bcarunmail/securing-rest-api-using-keycloak-and-spring-oauth2-6ddf3a1efcc2)
*   [Keycloak:从开发到生产的真实场景](https://medium.com/@siweheee/keycloak-a-real-scenario-from-development-to-production-ce57800e3ba9)
*   [如何设置 3scale OpenID Connect (OIDC)与 RH SSO 的集成](https://developers.redhat.com/blog/2017/11/21/setup-3scale-openid-connect-oidc-integration-rh-sso/)
*   [Red Hat Openshift 单点登录安全 N 层应用(@mechevarria)](https://github.com/aelkz/ocp-sso/blob/master/README.pt-br.md)

*Last updated: January 13, 2022*
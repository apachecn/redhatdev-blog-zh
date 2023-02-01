# 用 7 个简单的命令安装 Red Hat 3scale 并配置租户

> 原文：<https://developers.redhat.com/blog/2019/09/09/install-3scale-multitenant-in-7-commands>

几周前，我面临着安装 [Red Hat 3scale](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.6/) 并仅使用命令行配置其租户的挑战——不允许使用 GUI。这是一个相当有趣的用例，所以我决定写这篇文章，并展示如何只用七个命令来完成它！

(顺便说一下，我还决定在组合中包含 [Red Hat 单点登录](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.2/html/red_hat_single_sign-on_for_openshift/index) (SSO)，因为我希望我的 API 使用 [OpenID Connect](https://openid.net/connect/) (OIDC)进行认证。但是我将把这些命令留给以后的文章。)

## 要求

面对眼前的挑战，我知道如果我要成功，我必须充分利用 3scale Master API 和一些独创性。但是在进入解决方案之前，需要满足以下基本要求:

*   通过`oc` CLI 访问有权创建项目的 OpenShift 集群(或者至少为您创建两个项目，一个用于管理，一个用于每个租户)。
*   [3 个 scale OpenShift 模板](https://github.com/3scale/3scale-amp-openshift-templates)安装 3 个 scale 组件(`amp.yml`和`apicast.yml`)。
*   3 倍比例的图像和图像流。

> *免责声明:我使用的是带有 3scale v2.5 的 Red Hat open shift 3.11。open shift 4 . x 上 3scale (2.6+)的更新版本引入了使用 [3scale 操作符](https://github.com/3scale/3scale-operator)进行安装的选项，这与这里描述的完全不同。*

## 命令

所以，我们开始吧。让我们安装 3scale 和租户！

**注意:**在接下来的章节中，无论何时看到`${A_PLACEHOLDER_NAME}`，它都意味着需要用一个值来替换的东西，这个值可以是不言自明的，也可以是在前面的步骤中定义的。

### 命令#0:确保你已经掌握了基本知识

```
# the templates
git clone https://github.com/3scale/3scale-amp-openshift-templates
# the access with CLI
oc login ${YOUR_OCP_MASTER_CONSOLE_URL}
# the projects created
oc new-project 3scale-management-project
oc new-project 3scale-tenant-project
```

### 命令#1:安装 3 个规模(管理)

以下命令是安装 3scale 的最小命令。许多其他配置参数都是可用的，我强烈推荐阅读[安装指南](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.5/html/installing_3scale/onpremises-installation#deploying_3scale_on_openshift_using_a_template)。对于这个用例来说，这就足够了。

`MASTER_NAME (=3scale-master)`参数用作前缀来创建 3scale Master API 的 URL。在这种情况下，最终的 URL 应该类似于`3scale-master.apps.ocp.example.com`

```
oc new-app --file amp.yml \
  -p WILDCARD_DOMAIN=apps.ocp.example.com \
  -p MASTER_NAME=3scale-master \
  -n 3scale-management-project
```

**重要:**上面命令的输出将包含一些生成的令牌和秘密；记下它们。其余的命令将使用`MASTER_ACCESS_TOKEN`，但将来您可能需要其他值。

一旦上一个命令触发的安装完成，您就可以开始使用下一个命令创建租户。

### 命令#2:通过主 API 创建租户

下面的参数将使用变量`TENANT_NAME`来表示租户的名称，其他参数将从中派生出来。您应该用您想要的租户名称替换/定义它，例如， *devblog* 。

```
curl -X POST \
  https://3scale-master.apps.ocp.example.com/master/api/providers.xml \
  -d "access_token=${MASTER_ACCESS_TOKEN}" \
  -d "org_name=${TENANT_NAME}-tenant" \
  -d "username=${TENANT_NAME}-tenant-admin" \
  --data-urlencode "email=${TENANT_NAME}@example.com" \
  -d "password=${A_PASSWORD}"
```

主 API 应该返回一个成功的响应，其中包含新租户配置的 XML 表示。在有效负载中查找以下数据:

*   `TENANT_ACCESS_TOKEN`从元素的值`/signup/access_token/value`
*   `TENANT_ACCOUNT_ID`从元素的值`/signup/account/id`
*   `TENANT_USER_ID`从元素的值`/signup/users/user[0]/id`

### 命令#3:激活租户

默认情况下，新租户不会被创建为活动的。运行下面的命令激活它。

```
curl -X PUT \
  "https://3scale-master.apps.ocp.example.com/admin/api/accounts/${TENANT_ACCOUNT_ID}/users/${TENANT_USER_ID}/activate.xml" \
  -d "access_token=${MASTER_ACCESS_TOKEN}"
```

### 命令#4:创建租户的管理门户路由

该命令创建路由来配置网关(apicast ),以便它可以与其 API 管理器通信。

```
oc create route edge ${TENANT_NAME}-admin \
  --service=system-provider \
  --hostname=${TENANT_NAME}-tenant-admin.apps.ocp.example.com \
  --insecure-policy=Allow \
  -n 3scale-management-project
```

您可以使用命令#2 中提供的凭证(用户名/密码)通过这个新的 URL 访问租户的管理门户。

### 命令#5:使用 AMP 管理 URL 创建密码

```
oc secret new-basicauth apicast-configuration-url-secret \
  --password=https://${TENANT_ACCESS_TOKEN}@${TENANT_NAME}-tenant-admin.apps.ocp.example.com \
  -n 3scale-tenant-project
```

### 命令#6:安装租户的 API 网关(apicast)

以下命令仅安装 apicast 网关单元。由于该租户不会处理生产工作负载，因此它使用了为非生产部署推荐的最低配置参数。

```
oc new-app -f apicast.yml \
  -p CONFIGURATION_LOADER=lazy \
  -p DEPLOYMENT_ENVIRONMENT=staging \
  -p CONFIGURATION_CACHE=0 \
  -n 3scale-tenant-project
```

### 命令#7:公开您的租户的 API 网关

现在您只需要创建路由(URL)来将您的 apicast 服务(网关)公开给它的客户端。

```
oc create route edge \
  --service=apicast \
  --hostname=${TENANT_NAME}-tenant.apps.ocp.example.com \
  --insecure-policy=Allow \
  -n 3scale-tenant-project
```

您为这个租户创建的 API 将通过这个新的 URL 公开。

要创建更多的租户，只需重复命令#2 到#7。

## 结论

当我们向自动化迈进时，挑战自己不使用 GUI，而是用更难但更可重复的方式做事是很重要的。获得命令集只是自动化的第一步。接下来，我们可以将它们编译成脚本的形式，或者更好的，一个可翻译的剧本。这种功能提供的自动化不仅意味着可以轻松创建新的租户，还意味着可以从代码中重建整个环境。

尽管这些命令显示了一个 3scale 的简单配置，但要点是从现实生活中的咨询项目中提取的，当时每个租户代表软件交付生命周期的多个环境之一，如开发、测试、集成、预生产和生产。最终，这是一个更大图景的一部分，我们的客户需要将完整的 *API 生命周期作为代码*，这样它就可以自动化。

*Last updated: September 6, 2019*
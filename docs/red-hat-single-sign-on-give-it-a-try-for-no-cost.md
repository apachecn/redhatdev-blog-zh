# 红帽单点登录:免费试一试！

> 原文：<https://developers.redhat.com/blog/2019/02/07/red-hat-single-sign-on-give-it-a-try-for-no-cost>

在一个每天都比以前更加充满敌意的软件世界里，安全性很重要，开发人员正在处理越来越多的关于安全性的非功能性需求。最常见的是“OWASP 十大安全风险”:每个开发人员都应该知道的十大安全风险。还有许多你应该关心的安全风险，但是这十个风险是对你的软件安全影响最大的。其中包括认证和访问控制。

好消息是，由于 Red Hat Single Sign-On，身份验证和访问控制现在已经成为开源世界的商品。Red Hat Single Sign-On 是一个访问管理工具，它负责大多数身份验证协议的细节，如 SAML、OAuth 和 OpenID Connect 用户同意 UMA 甚至访问控制。它易于使用，有很好的文档记录，并且有一个非常活跃的社区:Keycloak。

本文描述了如何免费下载和安装 Red Hat 单点登录。

## 为什么不试一试？

Red Hat Single Sign-On 是 Red Hat Middleware Core Services Collections(以前称为 Red Hat JBoss Core Services Collection)的一部分，这意味着它与 [OpenShift 和 Red Hat middleware portfolio](https://www.redhat.com/en/resources/jboss-core-services-collection-datasheet) 的大多数产品一起提供支持。如果您已经在您的公司中使用这些产品，那么[您可能已经可以使用和部署 Red Hat 单点登录](https://access.redhat.com/articles/2294961)。

或者，作为开发人员，您也可以通过免费的开发人员订阅来尝试 Red Hat 单点登录。如果你已经有了开发者账号，进入[developers.redhat.com](https://developers.redhat.com/)，点击右上角的“登录”，登录。

## 注册开发者订阅

如果您还没有开发者订阅，请前往[developers.redhat.com/register](https://developers.redhat.com/register)创建您的帐户。

## 创建一个令牌来访问 Red Hat 注册表

您将需要创建一个令牌，以便能够从 Red Hat 注册表中获取 Red Hat 单点登录。转到[access.redhat.com/terms-based-registry](https://access.redhat.com/terms-based-registry/)，用你的开发者账户登录(如果你还没有这样做的话)，点击“新服务账户”

给令牌一个名称(在本文的其余部分，我们将使用“sso”)和一个有意义的描述。

单击“创建”,将显示生成的令牌。将用户名和令牌保存在安全的地方，以供将来参考。

[![ Generating a token](img/3d101ab68c8181f3a9d8fa879298f389.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/Screen-Shot-2019-02-01-at-16.21.31-1.png)

单击“OpenShift Secret”选项卡，然后单击“sso-secret.yaml ”,以 OpenShift 可以理解的格式下载您的令牌。把它保存在方便以后使用的地方。

[![Download your token](img/fc27702a147ea9214530923eb7000b3d.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/02/Screen-Shot-2019-02-01-at-16.22.50.png)

## 安装 Red Hat 单点登录

要安装 Red Hat 单点登录，您需要一个 OpenShift 实例。如果你的公司有，使用它。如果没有，我会推荐使用[红帽容器开发工具包(CDK)](https://developers.redhat.com/products/cdk/overview/) /minishift。 [Minishift](https://docs.okd.io/latest/minishift/getting-started/index.html) 是一款 OpenShift 安装软件，面向在笔记本电脑上运行的开发人员。如果需要安装 CDK/minishift，参见[这些说明](https://developers.redhat.com/products/cdk/hello-world/#fndtn-windows)。

加速运行一个 minishift 实例:

```
$ minishift start
```

为您的 Red Hat 单点登录部署创建一个新项目:

```
$ oc new-project sso
```

将之前下载的令牌作为一个秘密注入到 OpenShift 项目中:

```
$ oc create -f ~/Downloads/*_sso-secret.yaml
```

找到你的秘密的名字:

```
$ oc get secret
NAME                       TYPE                                  DATA      AGE
10072637-sso-pull-secret   kubernetes.io/dockerconfigjson        1         3m
```

如果您按照上面的建议将令牌命名为“sso”，那么您的秘密应该以“-sso-pull-secret”结尾在这个例子中，我的秘密被命名为“10072637-sso-pull-secret”

将您的令牌与默认服务帐户链接，以便此项目中的任何 pod 都可以使用它(不要忘记将“10072637-sso-pull-secret”更改为您的令牌名称):

```
$ oc secrets link default 10072637-sso-pull-secret --for=pull
```

并将 Red Hat 单一登录 7.3 映像导入到您的项目中:

```
$ oc import-image redhat-sso73-openshift:1.0 --confirm --scheduled --from=registry.redhat.io/redhat-sso-7/sso73-openshift:1.0
```

您现在可以部署 Red Hat Single Sign-On 7.3，如[文档](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/red_hat_single_sign-on_for_openshift/get_started)中所述。

您是否渴望看到 Red Hat 单点登录，但又不想阅读文档？以下是总结:

```
$ oc create -f https://raw.githubusercontent.com/jboss-container-images/redhat-sso-7-openshift-image/sso73-dev/templates/sso73-x509-https.json
$ oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default
$ oc new-app --template=sso73-x509-https -p SSO_ADMIN_USERNAME=admin -p SSO_ADMIN_PASSWORD=password -p IMAGE_STREAM_NAMESPACE=$(oc project -q)
```

等待几分钟，让 minishift 提取图像并启动 pod。您可以使用以下命令监督部署(按 Ctrl-C 退出):

```
$ oc get pods -w
```

部署后，使用以下命令显示控制台 URL:

```
$ oc get route sso -o jsonpath='http://{.spec.host}/auth/admin/'
```

在 web 浏览器中打开该 URL，并使用您在安装过程中提供的用户名和密码(在本例中为 admin/password)登录。

恭喜你；您刚刚安装了 Red Hat 单点登录！

## 进一步阅读

要从 Red Hat 单点登录中获得最大价值，请参阅[服务器管理指南](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/server_administration_guide/)和[安全应用和服务指南](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/securing_applications_and_services_guide/)。

您可能还希望使用带有 Red Hat 单点登录/Keycloak 的公共证书来阅读[。](https://developers.redhat.com/blog/2019/02/06/using-a-public-certificate-with-red-hat-single-sign-on-keycloak)

测试愉快！

*Last updated: September 3, 2019*
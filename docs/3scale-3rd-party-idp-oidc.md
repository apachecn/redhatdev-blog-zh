# 将第三方身份提供者与 Red Hat 3scale API 管理集成

> 原文：<https://developers.redhat.com/blog/2018/10/09/3scale-3rd-party-idp-oidc>

这篇文章描述了如何使用外部身份提供者(IdP)配置 OpenID 连接(OIDC)认证。随着[Red Hat 3 scale API Management](https://developers.redhat.com/products/3scale/overview/)版本 2.3 的新发布，在 API 认证阶段可以使用任何符合 OIDC 标准的 IdP。这是一个非常重要的新功能，因为它可以集成您环境中已经存在的任何 IdP，而不必使用身份代理，从而降低整体复杂性。

## 2.3 版之前的 API 身份验证

让我们回顾一下在这个版本发布之前 API 认证的可能性。

在 2.2 版及更高版本中，设置 OIDC 身份认证的典型配置场景需要三个组件:

1.  Red Hat 3scale API 管理(经理+网关)
2.  Red Hat 单点登录
3.  客户 IdP

配置完成后(您可以在这里找到更多细节)，最终用户应用程序将首先使用 Red Hat 单点登录对最终用户进行身份验证。这为客户 IdP 提供了接口，最终用户将在那里进行身份验证。在客户 IdP 和 Red Hat 单点登录之间还有一个令牌交换，这将向应用程序返回一个有效的访问令牌。然后，通过将令牌附加到标头并将请求转发到 3scale API 管理网关，应用程序将使用此令牌来访问受 3scale API 管理保护的 API 后端。网关将通过对访问令牌执行健全性检查来验证令牌，访问令牌需要是一个 **J** 儿子 **W** eb **T** oken (JWT)。(你可以在这里找到更多关于 OIDC 和 OAuth 的细节。)

为实际验证呼叫而执行的三个主要检查是:

1.  验证令牌是用 IdP 的公钥签名的(关于该过程的更多细节可在[此处](http://blog.differentpla.net/blog/2015/04/19/jwt-rs256-erlang)获得)
2.  检查令牌内的客户端 ID 是否与 3scale API 管理中的 ID 相对应
3.  验证令牌的 TTL(生存时间)尚未过期

这些检查基于符合 OIDC 标准的最低要求。

从版本 2.3 开始，只要客户 IdP 符合 OIDC 规范，上述流程将通过消除对 Red Hat 单点登录的需求而得到简化，如下所述。

## OIDC 合规

什么是 OIDC 合规？你可以在 OpenID 认证页面找到更多信息[。如你所见，红帽和钥匙钩在一起。该认证实施列表非常重要，因为有许多 IDP 实施了支持 OAuth2.0 认证场景的基础，但它们不符合更严格的 OIDC 标准。(在开始任何集成工作之前，了解这一点很重要。)](https://openid.net/certification/)

让我们以列表中提到的两个主要提供商为例，看看 3scale API Management 如何与他们合作。在以下场景中，我将使用:

1.  API 管理器组件的 My 3scale SaaS 帐户
2.  Docker 格式的本地运行 API 网关映像
3.  基于云的第三方 IdP
4.  运行在云中的 API 后端

对于最后一项，我将使用一个名为 [RandomUser](https://randomuser.me/) 的服务来生成随机用户数据，这在您用虚拟用户填充应用程序时非常有用。

## 与 Oracle IDCS 集成

Oracle 身份云服务是支持 OAuth2.0、OIDC 和 SAML 2.0 的在线 IdP。

我不会详细介绍该 IdP 的功能或如何注册 Oracle 云帐户，但请注意，从今天起，您可以注册免费试用。

首先，我将使用 Oracle 专家的一篇[文章来配置应用程序。](http://www.ateam-oracle.com/using-openid-connect-to-delegate-authentication-to-oracle-identity-cloud-service/)

让我们通过创建一个新的应用程序来开始配置 Oracle IDCS。

[![Oracle IDCS new application screen](img/c06a039f83f5fd196f68690666f0e937.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-11-51-14.png)

请注意，我将客户端配置为机密。记得在使用之前激活客户端。

然后，配置允许客户端访问的资源:

[![Audience section of the new application](img/d222ae83361b47965daf4d49bb73c4d2.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-11-52-39.png)

特别要注意主要受众配置，它包含客户端 ID 的值(我将在后面解释原因)。

将用户分配给应用程序:

[![Users assigned to the application](img/69a0164aeec710e3f0585f3ef1bb0961.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-12-02-08.png)

最后，不要忘记让访问 Oracle IDCS 公钥的端点可以公开访问。

[![Public key visibility](img/610c211010481a47395bc313e40bc249.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-12-07-25.png)

现在，我们来配置 3scale API 管理。从 API 认证的配置开始:

[![3scale API Management authentication settings](img/f5252db6d24fc401c8232fc1df710683.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-26-18-31-28.png)

然后，将 Oracle IDCS IdP 的端点指定为 OpenID 连接颁发者(作为参考，这应该是发现端点减去*)。知名/openid-configuration* 部分)。

[![3scale API Management IdP settings](img/accac9e9be8aeb72ce07e2cb7d8337cf.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-12-25-02.png)

(当您更新暂存配置时，请忽略验证错误，因为这与[动态客户端注册](https://openid.net/specs/openid-connect-registration-1_0.html#ClientRegistration)的功能相关，该功能仅适用于 Red Hat 单点登录)。我还必须在配置中添加一个头策略，因为我选择的后端服务似乎不接受长头，比如包含访问令牌的头。

最后，在 3scale API 管理上创建应用程序:

[![new application in 3scale API Management](img/22f043f23f72c36ea6d065a83388d298.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-12-32-54.png)

现在，让我们在我的机器上启动 3scale API 管理网关:

```
docker run --name apicast --rm -p 8082:8080 -e THREESCALE_PORTAL_ENDPOINT=https://*access_token*@lmf-admin.3scale.net -e APICAST_RESPONSE_CODES=true -e APICAST_SERVICES_LIST=2555417760770 -e APICAST_CONFIGURATION_CACHE=300 -e APICAST_LOG_LEVEL=info quay.io/3scale/apicast:master
```

如您所见，我正在过滤网关上的服务(以使故障排除更容易)，并且我正在使用最新版本的社区网关，因为该功能尚未发布。

现在让我们测试一下与 Postman 的完全集成:

[![OIDC authentication with Postman](img/c7d65aec2e2231fe38013127fb90796e.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-12-44-06.png)

一旦您通过了终端用户的身份验证，您将收到两个令牌:访问令牌和 ID 令牌。你可以在这里解码令牌的内容。通过比较这两个令牌可以看出，*和*声明的内容是不同的。在 ID 令牌的情况下，它还包含客户端 ID 的值。观众或 *aud* 一般表示谁可以消费服务，根据 OIDC 官方规格:

*aud*
必选。此 ID 标记面向的受众。它必须包含 *依赖方的 OAuth 2.0 client_id 作为受众值。它还可能包含其他受众的标识符。*

我们将使用 ID 标记，并将其添加到请求的报头中。

[![Response from Postman](img/eaf0e42bb1bb5b831eb557b5cf1d37af.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-13-06-02.png)

有用！

## 与 Microsoft Azure Active Directory 集成

我希望你还和我在一起。现在我们来看第二个积分。(如何设置 Azure 账户以及关于该产品的完整功能列表的信息不在本教程的讨论范围之内。)

我们将从在微软端创建应用程序开始。我将[使用这个微软页面作为起点](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-auth-code-flow)。

在继续这项工作之前，请注意以下几点:

*   我将使用 Active Directory 联合身份验证服务(ADFS)的端点版本 2，因为它们符合 OIDC 标准。
*   您将需要租户 ID，您可以通过导航到 Azure 门户内 Azure Active Directory 部分的属性窗格来找到它。
*   我将在 Microsoft 应用程序控制台上配置应用程序。

下面是我在 Azure 门户上创建的应用程序:

[![new application in the Microsoft portal](img/de48b3547dc3f59a5f76a82bf3806ab2.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-15-15-28.png)

我现在将在 3scale API 管理中修改服务配置:

[![3scale API Management IdP settings](img/eb8a8ad04b37064ba24e81d8d4f75cd8.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-15-19-22.png)

并创建一个新应用程序:

[![new application in 3scale API Management](img/f698585d36b89e2a74108ccafe1a4f0d.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-15-23-26.png)

我们将再次使用 Postman 来获取访问令牌。

[![OIDC authentication with Postman](img/66e32b73e8ed8a5c2cb6e484ec4cfd52.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/09/Screenshot-from-2018-09-27-15-26-10.png)

现在这里是实际的请求(再次使用 ID 令牌，原因与上面解释的相同)。

![response from Postman](img/f61ea3d48ced00fc216be34eb16bc5b5.png)

太好了！谢谢你陪我看完了与 OIDC 相关的所有技术细节。

请继续关注更多关于 API 认证和授权的博文！

*Last updated: September 3, 2019*
# 3 使用 OpenID Connect 通过 Red Hat 与 ForgeRock 的集成进行扩展

> 原文：<https://developers.redhat.com/blog/2018/03/12/3scale-red-hat-integration-forgerock-using-openid-connect>

在我的[上一篇文章](https://developers.redhat.com/blog/2018/03/06/redhat-3scale-identity-management/)中，我写了 API 管理和身份管理如何以互补的方式协同工作，以保护和管理应用程序作为 API 公开的服务/端点。在那篇文章中，我介绍了如何使用 Red Hat 3scale API 管理来集成身份管理器，此外还提供了 API 管理功能，如速率限制和节流。

本文将展示如何通过 Red Hat 将 [ForgeRock](https://www.forgerock.com/) 与 3scale 集成。ForgeRock 是一家受欢迎且不断发展的身份管理公司。ForgeRock 帮助组织安全地与客户、员工、设备和事物进行交互。

![](img/57bfd8957f048a51be515a90c60abfca.png)

对于本教程，将使用以下安装程序:

*   [锻造 OpenAM 13.0.0](https://backstage.forgerock.com/downloads/)
*   [3 scale API cast API gateway v 3.0](https://github.com/3scale/apicast)
*   3 规模 SAAS API 管理器

以下是组件:

![](img/c7ff1423025e1e25526f97bb0ea5c1f9.png)

**工作流程**

1.  客户端应用程序向 APIcast API 网关发送带有所需请求参数的请求。
2.  APIcast 使用 API 管理器验证凭证，如果有效，则存储在缓存中。
3.  APIcast 将请求发送到 ForgeRock，在那里对用户进行身份验证并获得最终用户的同意/授权。
4.  ForgeRock 用一个`id_token`将最终用户发送回客户端，如果请求的话，还会发送一个`access_token`。
5.  对于每个 API 调用，JWT 令牌通过 APIcast 发送到 API 后端，在那里用伪造的公钥*验证传入的`JWT`。如果有效，则将调用代理到 API 后端。*
6.  后端 API 提取 JWT，验证用户的范围，并将 API 响应发送回客户端应用程序。

**序列图**

![](img/b8dc19c42e5e563d2ce4a308ab2bd9b2.png)

为了完成端到端的集成，我们应该逐个设置所有部分。下面是组件和说明。

### **设置 API 后端**

对于这个演示，我将使用由 3scale by Red Hat 托管的 [echo API](https://echo-api.3scale.net/) 服务。您总是可以编写一个服务来提取 JWT、解析 JSON 有效负载、提取用户配置文件并发回该用户的产品订阅状态。

### **设置 API 管理器**

1.  通过 Red Hat 管理门户登录 3scale。
2.  选择要用于启用 OpenId Connect 与 ForgeRock 集成的服务。点击**API**选项卡，选择*服务*，点击**集成**链接。我们使用默认的 *Echo API:*

![](img/01b8b1306e5b1095057f4798265a06bf.png)

3.点击`edit integration settings`:

![](img/1af66cd49493bc0e3b42ee707f4157f7.png)

4.选择`OpenID Connect`并点击`Update Service`:

![](img/b431cddc7f656149bbd2c6dfd7f270fd.png)

5.返回整合页面，点击`edit APIcast configuration`:

![](img/7e615a8e15552a76c6293f52635a1834.png)

6.输入`Staging and Production base URL`。我们将在 docker 上本地部署 APIcast 网关，因此将其命名为`http://localhost:8080`:

![](img/3d9c9b3537b4f849f441bbdcf228d9d3.png)

7.最后点击`Update Staging Environment`。您还可以将其提升到生产环境(可选)。

8.创建一个`application`并获得`client_id`和`client_secret`。

8.1 转到开发者选项卡并点击`Developers`:

![](img/226eb6f2e6199bc0c4dd95595bf7ce66.png)

8.2 点击`Application`:

![](img/1681da152e0babcdeadac91340f408f1.png)

8.3 点击`Create Application`链接:

![](img/f32813da1e14f04b68ef680c0ef85c70.png)

8.4 选择服务的`Application Plan`，然后点击`Create Application`:

![](img/d3b0755df16daf531560c01f6d6b5e02.png)

8.5 记下`client_id`和`client_secret`。我们将使用 Postman 来测试我们的集成，因此我们将使用一个固定链接来填充回调信息。键入“https://www . get postman . com/oauth 2/callback”。在“重定向 URL”字段中。单击更新按钮。

![](img/917ed71356f6f2250b88dc3afcb4ba7d.png)

仅此而已！现在让我们来看看**锻石**的场景。

**搭建假山**

ForgeRock 的安装超出了本教程的范围。请参考岩石文件进行安装。安装 ForgeRock 后，确保你可以访问 http://openam.mydomain.com:8080/openam 上的网址。

1.  创建`Realm`:

 ![](img/46fb718ffe13f08340bc359d9b0a75a2.png)

2.点击`Configure Oauth Provider` → `Configure OpenID Connect`:

 ![](img/6ac2c75b0c510f827371a6143e4babe5.png)

3.点击`Create`:

 ![](img/62888a359a21eafe4ea26ba731bedf66.png)

4.用 ForgeRock 创建(或同步)3scale by Red Hat `client_id`。

> 我们的首席开发人员 [Michal Cichra](https://github.com/mikz) 编写了一个名为 [Zync](https://github.com/3scale/zync) 的工具，将所有 3 个 scale by Red Hat client_ids 同步到 IDP。因此，每次创建应用程序时(即 Red Hat 的 3scale 上的`client_id`和`client_secret`)，IDP 端都会自动创建相同的应用程序。在本练习中，我使用下面的注册信息手动创建了 client_ids。如果您更喜欢创建 ids 运行时，请使用 ForgeRock 的客户端注册端点来编辑该工具。欢迎公关。

4.1 点击`Agents` → `Oauth2.0/OpenID Connect Client` → `New`:

 ![](img/5a877b2be9a370d8b3043f87aac4a822.png)

4.2 从您之前创建的管理门户复制 3scale by Red Hat `client_id`和`client_secret`。输入`Name`为`client_id`，输入`Password`为`client_secret`。点击**创建:**

![](img/3943b9daa73de3c2ebccf066bc490a56.png)

4.3 输入 https://www.getpostman.com/oauth2/callback 的`Redirection URIs` → [和`Scope` → openid。点击**保存:**](https://www.getpostman.com/oauth2/callback)

 ![](img/e1e7b649edbf800918a5345427eab71e.png)

5.创建将根据 IDP 进行身份验证的**最终用户**。

5.1 转到`Realms` → `Subjects.`点击**新建:**

 ![](img/1af7a228e54106baf4e76ae17fa37121.png)

5.2 输入“ID: apiUser”和“密码:12345678 ”:

 ![](img/60ebdab62a346c2d8c45909f432020f6.png)

一切准备就绪！

**设置 APIcast API 网关**

确保在执行下一个命令之前安装 *docker* 和 *docker-compose* 。我们将在本地运行 APIcast API gateway，它将接受来自客户端的所有传入请求。

```
1\. git clone git@github.com:VinayBhalerao/3scale-forgerock-integration.git
2\. Edit the .env file per your setup
3\. docker-compose up

```

### **向 APIcast 发送请求**

1.  向 APIcast 发送授权请求

```
GET http://localhost:8080/authorize?client_id=21657b2d&scope=openid&response_type=token id_token&nonce=1234&redirect_uri=https://www.getpostman.com/oauth2/callback&realm=internal
```

```
where,
```

```
client_id = 3scale client id
scope = openid
response_type = token id_token
nonce = 1234
redirect_uri = https://www.getpostman.com/oauth2/callback
realm = internal
```

2.从 Forgerock 显示一个登录页面。输入我们之前为最终用户创建的凭据:

输入凭据为:apiUser / 12345678:

![](img/95bbe979cee1370ef669323ca3c333d7.png)

点击**允许:**

![](img/8ce4a7ac7e4c9f70d607d8e4ba018479.png)

一个`access_token`和`id_token`被重定向回应用程序。`id_token`是由 IDP 生成的 JWT 令牌。

在 JWT.io 网站上粘贴令牌以解密内容(可选):

![](img/0360c3dc4634e83aab8cc36eb710440f.png)

对于每个调用，上述令牌被发送到 APIcast 网关。网关将使用*公钥*来验证 JWT 的签名。如果有效，该调用将与 JWT 一起被代理到 API 后端。然后，后端负责对 JWT 进行 base64 解码，从 JSON 有效负载中提取用户配置文件，然后(根据配置文件)发回 API 响应。[请参考[微服务如何验证 JWT](https://nordicapis.com/how-to-control-user-identity-within-microservices/#comment-3637019878) 了解更多详情]

**来自 APIcast 的请求和响应:**

```
curl -H "Authorization: Bearer eyAidHlwIjogIkpXVCIsICJraWQiOiAiU3lsTEM2Tmp0MUtHUWt0RDlNdCswemNlUVNVPSIsICJhbGciOiAiUlMyNTYiIH0.eyAiYXRfaGFzaCI6ICJUcUd5c2dRZURsWDhIOFNHR1FjcEF3IiwgInN1YiI6ICJhcGlVc2VyIiwgImlzcyI6ICJodHRwOi8vdmJoYWxlcmEub3NlY2xvdWQuY29tOjgwODAvb3BlbmFtL29hdXRoMi9pbnRlcm5hbCIsICJ0b2tlbk5hbWUiOiAiaWRfdG9rZW4iLCAibm9uY2UiOiAiMTIzNCIsICJhdWQiOiBbICIyMTY1N2IyZCIgXSwgIm9yZy5mb3JnZXJvY2sub3BlbmlkY29ubmVjdC5vcHMiOiAiNjk2YmRlNTYtZmNiZi00ZTFkLWIzOGItYmMzNzQ4OGVhODRiIiwgImF6cCI6ICIyMTY1N2IyZCIsICJhdXRoX3RpbWUiOiAxNTE2OTE3NTQwLCAicmVhbG0iOiAiL2ludGVybmFsIiwgImV4cCI6IDE1MTY5MjEyMzUsICJ0b2tlblR5cGUiOiAiSldUVG9rZW4iLCAiaWF0IjogMTUxNjkxNzYzNSB9.SuYI1tP5uJ94y8XRc6QQClXlmuLzMFEcE1LlW_31GafXv91jg3QwbRI-1RV1XOISfWnLW7l-1eGyKZtK_P8nroLjXYs2c-HrIgTwK16FBTcM9-Gt_jzbntwN4hiLD4PbhVb562fTkdqQCA4ZlNR9QOmQUE0ZKlMSwB3b0bNSmys" http://localhost:8080/subscriptions -v
*   Trying ::1...
* Connected to localhost (::1) port 8080 (#0)
> GET /subscriptions HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.43.0
> Accept: */*
> Authorization: Bearer eyAidHlwIjogIkpXVCIsICJraWQiOiAiU3lsTEM2Tmp0MUtHUWt0RDlNdCswemNlUVNVPSIsICJhbGciOiAiUlMyNTYiIH0.eyAiYXRfaGFzaCI6ICJUcUd5c2dRZURsWDhIOFNHR1FjcEF3IiwgInN1YiI6ICJhcGlVc2VyIiwgImlzcyI6ICJodHRwOi8vdmJoYWxlcmEub3NlY2xvdWQuY29tOjgwODAvb3BlbmFtL29hdXRoMi9pbnRlcm5hbCIsICJ0b2tlbk5hbWUiOiAiaWRfdG9rZW4iLCAibm9uY2UiOiAiMTIzNCIsICJhdWQiOiBbICIyMTY1N2IyZCIgXSwgIm9yZy5mb3JnZXJvY2sub3BlbmlkY29ubmVjdC5vcHMiOiAiNjk2YmRlNTYtZmNiZi00ZTFkLWIzOGItYmMzNzQ4OGVhODRiIiwgImF6cCI6ICIyMTY1N2IyZCIsICJhdXRoX3RpbWUiOiAxNTE2OTE3NTQwLCAicmVhbG0iOiAiL2ludGVybmFsIiwgImV4cCI6IDE1MTY5MjEyMzUsICJ0b2tlblR5cGUiOiAiSldUVG9rZW4iLCAiaWF0IjogMTUxNjkxNzYzNSB9.SuYI1tP5uJ94y8XRc6QQClXlmuLzMFEcE1LlW_31GafXv91jg3QwbRI-1RV1XOISfWnLW7l-1eGyKZtK_P8nroLjXYs2c-HrIgTwK16FBTcM9-Gt_jzbntwN4hiLD4PbhVb562fTkdqQCA4ZlNR9QOmQUE0ZKlMSwB3b0bNSmys
>
< HTTP/1.1 200 OK
< Server: openresty/1.11.2.2
< Date: Thu, 25 Jan 2018 22:03:31 GMT
< Content-Type: application/json
< Content-Length: 1480
< Connection: keep-alive
< Cache-control: private
< Set-Cookie: d8c1dd0e39ac4456ed39ce5889b9a5a5=e3380f4380dfce29d71b1a31cd3dd973; path=/; HttpOnly
< Vary: Origin
< X-Content-Type-Options: nosniff
<
{
  "method": "GET",
  "path": "/subscriptions",
  "args": "",
  "body": "",
  "headers": {
    "HTTP_VERSION": "HTTP/1.1",
    "HTTP_HOST": "echo-api.3scale.net",
    "HTTP_ACCEPT": "*/*",
    "HTTP_AUTHORIZATION": "Bearer eyAidHlwIjogIkpXVCIsICJraWQiOiAiU3lsTEM2Tmp0MUtHUWt0RDlNdCswemNlUVNVPSIsICJhbGciOiAiUlMyNTYiIH0.eyAiYXRfaGFzaCI6ICJUcUd5c2dRZURsWDhIOFNHR1FjcEF3IiwgInN1YiI6ICJhcGlVc2VyIiwgImlzcyI6ICJodHRwOi8vdmJoYWxlcmEub3NlY2xvdWQuY29tOjgwODAvb3BlbmFtL29hdXRoMi9pbnRlcm5hbCIsICJ0b2tlbk5hbWUiOiAiaWRfdG9rZW4iLCAibm9uY2UiOiAiMTIzNCIsICJhdWQiOiBbICIyMTY1N2IyZCIgXSwgIm9yZy5mb3JnZXJvY2sub3BlbmlkY29ubmVjdC5vcHMiOiAiNjk2YmRlNTYtZmNiZi00ZTFkLWIzOGItYmMzNzQ4OGVhODRiIiwgImF6cCI6ICIyMTY1N2IyZCIsICJhdXRoX3RpbWUiOiAxNTE2OTE3NTQwLCAicmVhbG0iOiAiL2ludGVybmFsIiwgImV4cCI6IDE1MTY5MjEyMzUsICJ0b2tlblR5cGUiOiAiSldUVG9rZW4iLCAiaWF0IjogMTUxNjkxNzYzNSB9.SuYI1tP5uJ94y8XRc6QQClXlmuLzMFEcE1LlW_31GafXv91jg3QwbRI-1RV1XOISfWnLW7l-1eGyKZtK_P8nroLjXYs2c-HrIgTwK16FBTcM9-Gt_jzbntwN4hiLD4PbhVb562fTkdqQCA4ZlNR9QOmQUE0ZKlMSwB3b0bNSmys",
    "HTTP_USER_AGENT": "curl/7.43.0",
    "HTTP_X_3SCALE_PROXY_SECRET_TOKEN": "secret_token_vinay_demo",
    "HTTP_X_REAL_IP": "172.21.0.1",
    "HTTP_X_FORWARDED_FOR": "76.102.119.200, 10.0.103.186",
    "HTTP_X_FORWARDED_HOST": "echo-api.3scale.net",
    "HTTP_X_FORWARDED_PORT": "443",
    "HTTP_X_FORWARDED_PROTO": "https",
    "HTTP_FORWARDED": "for=10.0.103.186;host=echo-api.3scale.net;proto=https"
  },
  "uuid": "4b100977-4b31-4dc7-9b45-bf5dadb50d97"
* Connection #0 to host localhost left intact
```

感谢您抽出时间阅读本教程。在我的下一篇博文中，我将介绍如何使用 OpenID Connect 将 3scale by Red Hat 与 pingblean 集成在一起。

*Last updated: March 13, 2018*
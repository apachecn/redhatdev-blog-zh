# 使用 SAML 将 RH-SSO 7.x 与 Liferay DXP 集成

> 原文：<https://developers.redhat.com/blog/2018/02/01/rh-sso-liferay-dxp-saml>

本教程的目的是配置[Red Hat Single Sign On](https://access.redhat.com/products/red-hat-single-sign-on)(RH-SSO)以通过 SAML 作为 Liferay DXP 的身份提供者(IdP)。

Liferay DXP 支持单点登录(SSO)功能，如 NTLM、OpenID 和基于令牌的功能，以及与谷歌和脸书等 IdPs 的集成。但是当涉及到企业环境时，要求可能会更严格，尤其是关于与外部 IDP 的集成。

Red Hat Single Sign On (RH-SSO)是 [Keycloak](http://www.keycloak.org/) 的企业版，可以在对外部 IDP 有限制的公司中作为身份提供者。使用 RH-SSO 进行身份管理是一个常见的场景，由 LDAP 服务器提供支持。在这种情况下，其他公司的应用程序将需要与 RH-SSO 集成，以便在整个环境中实现单点登录功能。

实现与 RH-SSO 集成的一种方式是使用 [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) ，这是一种用于在两个参与者之间交换认证和授权数据的开源标准。交换的数据是 XML 格式的，包含使用安全连接(HTTPS)用证书加密的用户信息。幸运的是，Liferay DXP 提供了与 SAML 的集成。

## 常见用例

以下是举例说明 Liferay DXP 和 RH-SSO 之间的一个样本认证用例的总结流程。

1.  用户请求公司门户上的一个页面。
2.  门户发现用户没有经过身份验证，并将用户重定向到身份提供者，在本例中是 RH-SSO。
3.  RH-SSO 通过为其配置的方法之一对用户进行身份验证(基本身份验证、表单、Kerberos 等)。
4.  用户被重定向回带有 SAML 数据的门户。
5.  最后，门户使用 SAML 数据根据用户的信息(电子邮件、屏幕名称、全名、组等)对用户进行授权。).如果是新用户，门户会在自己的数据库中创建用户。

## 使用 SAML 作为 SSO 机制的步骤

第一步是将 RH-SSO 设置为 SAML 身份提供者，然后在用户通过身份验证后映射要调度的用户数据。

最后一步是将 Liferay DXP 公司设置为 SAML 服务提供商(SP)。Liferay 的官方文档解释了如何做到这一点(在本指南中用作参考)。

### 将 Red Hat 单点登录配置为 SAML 身份提供者

在这个阶段，预计会有一个预配置了 LDAP 或其他用户联盟的全功能 RH-SSO 服务器。本 RH-SSO 安装指南可用于该任务。

按照以下步骤将 RH-SSO 设置为 SAML 身份提供者。

1)登录 RH-SSO 管理控制台。在左侧菜单中的“配置”部分，单击“客户端”。

2)单击“创建”并在“客户端 ID”属性中填入稍后将用于配置 Liferay 门户的 ID。在“客户端协议”属性中选择“saml”选项。下图说明了这一过程。

[![](img/1204566c6e0d3a5ebedf89f8c58756b1.png "create_client_upload_xml")](/sites/default/files/blog/2018/01/create_client_upload_xml.png)

Add a new SAML Client

3)单击“保存”以创建新客户端，此时会出现更多选项。默认选项在标准配置中应该可以正常工作。当 Liferay DXP 配置完成时，来自服务提供商的重定向 URL 稍后被更新。

4)点击“安装”选项卡，选择“SAML 元数据 IDPSSODescriptor”选项并下载。此文件包含有关 RH-SSO 身份提供者的信息，应该用于与 Liferay DXP 链接。

[![](img/ff6e77531d80468641c9f9198663b174.png "rhsso-download_ssoidp_xml_file")](/sites/default/files/blog/2018/01/rhsso-download_ssoidp_xml_file.png)

Download SAML Metadata IDP SSO Descriptor

### 将 RH-SSO 用户模型映射到 SAML 客户机

Liferay DXP 需要登录用户的数据，如电子邮件、名字、姓氏等。为了将用户模型从 RH-SSO 映射到 SAML 客户机，有必要提供这些信息。在 RH-SSO 认证用户之后，用户信息通过 XML 传递给 Liferay。

以下说明要求已经在 RH-SSO 中设置了 LDAP 用户联盟。在 RH-SSO 手册的[用户存储联盟](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.1/html/server_administration_guide/user-storage-federation)部分中有关于此过程的详细说明

下面解释了将用户属性从 LDAP 映射到 SAML 客户端的过程。按照以下步骤创建映射器，这些映射器需要在认证过程中提供 Liferay DXP 所需的用户属性。至少，应该提供 username 属性以使流程正常工作。

1)将 LDAP 映射器添加到用户联盟中的属性。对于某些属性，已经创建了一个映射器(姓氏、名字和其他一些基本属性)。在控制台中的用户联盟下，选择 LDAP 提供程序，然后选择 Mappers 选项卡。

为所有必需的 ldap 属性创建新的用户属性 LDAP 映射程序。例如，电子邮件属性是这样创建的:

**名称**:电子邮件
**类型**:用户-属性-ldap-mapper
**用户模型属性**:电子邮件(RH-SSO 中的属性名称)
**ldap 属性**:邮件(LDAP 中的属性名称)
**只读**:开/关
**总是从 LDAP 中读取值**:开/关
**在 LDAP 中是强制的**

[![](img/29b668005a17e62c4a62b2e5e6385a75.png "rhsso-ldap-federation-email-mapper")](/sites/default/files/blog/2018/01/rhsso-ldap-federation-email-mapper.png)

LDAP Mappers - add email mapper

2)添加协议映射器。映射器将属性从 RH-SSO 用户模型移动到 SAML 协议断言。在控制台中的客户端下，选择 SAML 客户端，然后选择映射器。添加用户属性映射器。按照之前的电子邮件示例:

[![](img/8b172f03e7a0f45eda5e372b21d9a7f2.png "rhsso-mappers_tab_saml_client")](/sites/default/files/blog/2018/01/rhsso-mappers_tab_saml_client.png)SAML client mappers tab

SAML Client Mappers tab

**名称** : email
**需要同意** : OFF
**映射器类型**:用户属性
**属性** : email(这是之前映射器中的“用户模型属性”，RH-SSO 中的属性名称)
**友好名称** : email
**SAML 属性名称** : email
**名称格式**:基本

[![](img/8e73a2ba2299f5bc37c5b2ae3a03c9ea.png "rhsso-saml-client-provider-username-mapper")](/sites/default/files/blog/2018/01/rhsso-saml-client-provider-username-mapper.png)

SAML Protocol Mapper example

### 将 Liferay DXP 设置为 SAML 服务提供商

现在是时候配置 Liferay 来执行服务提供者角色了。以下是来自 [Liferay 官方文档](https://dev.liferay.com/es/discover/portal/-/knowledge_base/6-2/integrating-existing-users-into-liferay?_ga=2.126361739.700801911.1510231190-50392736.1504544452#setting-up-liferay-as-a-saml-service-provider)的一些步骤的总结。

1)使用控制面板的市场界面或手动安装 [SAML 2.0 Provider EE 应用程序](https://web.liferay.com/marketplace/-/mp/application/15188711)。

2)通过在“SAML 管理”菜单上选择此选项，将 Liferay 设置为服务提供商 SAML 角色。输入服务提供商名称，然后单击“保存”。

[![](img/84ff50173cd98e9e3b74a5787f8994a9.png "liferay-saml-general-tab")](/sites/default/files/blog/2018/01/Screenshot_20180111_141507.png)

Liferay DXP SAML Provider - Admin General Tab

标题为“**证书和私钥”**的新部分出现。

3)证书和私钥部分允许您输入用于为 SAML 创建密钥库的信息。

输入以下信息:

*   常用名(可能是管理员的名和姓)
*   组织名称
*   组织单位的名称
*   城市或地区
*   州或省
*   国家
*   密钥库保持有效的天数(密钥库过期前的时间)
*   密钥算法(RSA 是默认算法)
*   以位为单位的密钥长度(默认值为 2048)
*   密钥密码

输入完所有需要的信息后，点击*保存*。

[![](img/1cf956d175ee96314a7c0d1789aff6dd.png "liferay-saml-add-certificate")](/sites/default/files/blog/2018/01/Screenshot_20180111_141713.png)

Liferay DXP SAML Provider - Add a new certificate and private key

4)现在可以查看证书信息或下载证书。下载它，如果没有失败，密钥库创建就成功了。创建密钥库后，其他选项会出现在 SAML 管理控制面板中:

*   一般
*   互联网服务商
*   身份提供者连接

5)使用上一步下载的证书作为输入来完成 RH-SSO 配置。请将其保存在安全的位置。

### 连接救生筏 DXP 和 RH-SSO

通过 SAML 完成 Liferay DXP 和 RH SSO 之间的集成所需的最后一项配置是在“SAML 管理”配置页面上定义 IdP 选项:

1)登录 Liferay DXP 并点击“SAML 管理”菜单。

2)单击“身份提供者连接”选项卡，并输入您的配置名称。

3)在“EntityID”字段输入 RH-SSO 领域 URL，例如:**http://rhsso.mycorporate.com/auth/realms/myrealm**。该信息由从 RH-SSO 下载的 XML 在“entityID”属性中提供。

[![](img/779629bcae66c22d0caea4683de1eb83.png "liferay-dxp-saml-general-tab-entityid")](/sites/default/files/blog/2018/01/Screenshot_20180111_141555.png)

Liferay DXP SAML Provider - Add IdP entity ID

4)将“名称标识符”保留为“未指定”。

5)上传上一节“将 Red Hat 单点登录配置为 SAML 身份提供者”中从 RH-SSO 下载的元数据 XML 文件。

6)在“属性”部分，将 Liferay 用户属性映射到在前面部分“将 RH-SSO 用户模型映射到 SAML 客户端”中创建的属性。在那里创建的“SAML 属性名”字段是映射期间应该使用的属性名。

Liferay 属性有:emailAddress、screenName、firstName、lastName 和 uuid。

确保至少映射屏幕名称和电子邮件属性，因为 Liferay 需要在身份验证过程中创建用户。格式是 SAML 属性后跟 Liferay 的。每行一个属性:

```
username=screenName
email=emailAddress

```

[![](img/c4d5f11b881229ccd65582acf743b512.png "liferay-saml-provider-attributes-mappers")](/sites/default/files/blog/2018/01/Screenshot_20180111_141617.png)

Liferay DXP SAML Provider - Attributes mapping

```
7)点击“保存”以更新配置。
8)返回“常规”选项卡并启用 SAML(点击“**启用**复选框)。
9)登录 RH-SSO 并转到客户端、已配置的 Liferay SAML 客户端，然后转到 SAML 密钥。点击“导入”并导入之前从 Liferay 门户下载的证书。检查证书是否已正确更新:

  [![](img/e5d2b4d750977360f46f1ca8910085e9.png "rhsso-saml-provider-saml-keys")](/sites/default/files/blog/2018/01/Screenshot_20180111_142319.png)

RH-SSO SAML Provider certificate keys

执行完这些步骤后，配置应该可以使用了。若要验证，请尝试注销并再次登录。Liferay 应该将用户重定向到 RH-SSO 认证机制，然后返回到已经通过认证的门户。
SAML 集成故障排除
测试这种集成的第一件事是建立一个清晰的环境，通过网络连接到 Liferay DXP 和 RH-SSO。之后，按照下面的过程来解决 SAML 配置过程中可能出现的常见问题。
认证后，我从 RH-SSO 控制台收到“重定向错误”
检查在 RH-SSO 控制台上的“客户端”选项卡中是否正确映射了 URL。主机名必须与 Liferay 和通过浏览器访问的 URL 相匹配。另外，检查 RH-SSO 是否可以访问指定的主机名。**提示**:如果服务器不在同一个位置，作为重定向 URL 的“localhost”将不起作用。
在向 RH-SSO 请求 SAML 的过程中，我遇到了消息标志问题
检查 Liferay 的证书是否已成功导入 RH-SSO 客户端配置。请参见“SAML 密钥”选项卡；证书字段必须包含来自 Liferay 的证书文件。在文本编辑器中打开证书文件，检查内容是否与该字段中的内容匹配。
Liferay 抱怨在新用户认证过程中需要填写的字段
另一个常见问题是 Liferay 抱怨在 SAML 解析过程中缺少用户属性。Liferay 的日志显示如下消息:

```
17:42:07,023 ERROR [http-nio-8080-exec-4][BaseSamlStrutsAction:46] com.liferay.portal.kernel.exception.ContactNameException$MustHaveFirstName: Contacts must have a first name
```

根据本文中的**将 RH SSO 用户模型映射到 SAML 客户端(添加锚点)**部分，检查 RH-SSO 中的 SAML 映射器属性是否定义正确。此外，检查 RH-SSO 中的用户模型是否有 Liferay 抱怨的缺失属性。例如，LDAP 用户是否有关联的电子邮件？
最后，验证 Liferay SAML 设置中的属性映射器是否正确。它是由等号(=)分隔的键值对，每行一个:“username=screenName”。关键首先是 SAML 属性，其次是 Liferay 的用户属性。
要调试 SAML 响应，请查看本文[。使用](http://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_saml_view-saml-response.html)[这个在线工具](https://www.samltool.com/decode.php)解码 SAML 响应，并检查用户属性。
参考
1.  [根据 Liferay 的官方文档将 Liferay 设置为 SAML 服务提供商](https://dev.liferay.com/es/discover/portal/-/knowledge_base/6-2/integrating-existing-users-into-liferay?_ga=2.126361739.700801911.1510231190-50392736.1504544452#setting-up-liferay-as-a-saml-service-provider)
2.  [OIDC 令牌和 SAML 断言映射](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.1/html/server_administration_guide/clients#protocol-mappers)
3.  [LDAP/联盟映射器](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.1/html/server_administration_guide/user-storage-federation#ldap_mappers)来自 Red Hat 单点登录手册
4.  [如何在红帽单点登录 7.x 中将 LDAP 用户联盟的用户属性映射到 SAML 客户端？](https://access.redhat.com/solutions/3244771)
5.  [如何在浏览器中查看 SAML 响应以排除故障](http://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_saml_view-saml-response.html)

```
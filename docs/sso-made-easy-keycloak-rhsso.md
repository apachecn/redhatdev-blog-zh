# 使用 Keycloak / Red Hat SSO 简化单点登录

> 原文：<https://developers.redhat.com/blog/2018/03/19/sso-made-easy-keycloak-rhsso>

如果您正在寻找一个单点登录解决方案(SSO ),使您能够保护新的或传统的应用程序，并轻松地使用联合身份提供者(IdP ),如社交网络，您肯定应该看看 Keycloak。Keycloak 是红帽单点登录(RH-SSO)的上游开源社区项目。RH-SSO 是一个核心服务，是 Red Hat JBoss Enterprise Application Platform 等许多产品的一部分。如果你已经登录到[developers.redhat.com](https://developers.redhat.com/)或[openshift.com](https://www.openshift.com/)，你正在使用 Keycloak。

在 Red Hat 开发人员的博客上，有许多最近的文章，涵盖了 Keycloak/RH-SSO 集成的各个方面。最近的一次 [DevNation Live](https://developers.redhat.com/devnationlive/) 技术讲座涉及了[用 Keycloak](https://developers.redhat.com/blog/2018/02/28/devnation-live-secure-spring-boot-keycloak/) 保护 Spring Boot 微服务。本文讨论了您应该了解的 Keycloak/RH-SSO 的特性。

When considering any SSO platform, there are six aspects to consider:

1.  适应性
2.  综合
3.  可量测性
4.  展开性
5.  集中
6.  特征

### 适应性

By default, Keycloak uses the open source [H2 database](http://www.h2database.com) as its embedded datastore. However, you are free to choose your own database: Oracle, Microsoft SQL Server, IBM DB2 , MySQL/MariaDB, or PostgreSQL.

### 综合

Would you like to enable social networking logins for your application?  You can configure your Keycloak instance to use OpenID providers like: Google, Facebook, Twitter, Github, LinkedIn, Microsoft, or StackOverflow.For enterprise deployments, Keycloak supports Kerberos logins. And yes, you can also federate your LDAP or Active Directory in just one configuration page.Keycloak is easy to add to your application as adapters are available for many platforms including: JBoss EAP, JBoss Fuse, JavaScript, Node.js, and mod_auth_mellon for Apache.

### 可量测性

Keycloak use the Wildfly Clustering features which means you can use Infinispan (aka Red Hat JBoss DataGrid) caching to have clustered instances with sessions replicated across all the machines in the cluster.

### 展开性

The Keycloak Service provider interface, enables you to write your own authenticator or federator. You could use the authenticator, to integrate your own code that authenticate a user with SSL client certificate. (Though this is now a built-in feature starting with version 3.2). Using the federator, you could integrate the user database from a legacy system and use it as an identity source in Keycloak.

### 集中

KeyCloak has centralized session management.  The benefits are:

*   您可以确定系统当前有多少个活动会话。
*   您可以强制单个用户注销。
*   或者，您可以强制系统的所有用户注销。

*Logout All Limitations: Any SSO cookies set will be invalid and clients that request authentication in active browser sessions will now have to re-login. Only certain clients are notified of this logout event, specifically clients that are using the Keycloak OIDC client adapter. Other client types (i.e. SAML) will not receive a backchannel logout request.*

### 很酷的功能

*   一次性密码(OTP)策略
*   集中式密码策略
*   每个资源或每个范围的授权策略
*   定时访问策略(用户或用户组只能在特定时间段登录)
*   基于 JavaScript 的策略
*   基于规则的策略

### Keycloak 入门

[![](img/49d550130db600c7ef8c4b51211b13c2.png "keycloaklogin-page")](/sites/default/files/blog/2018/03/keycloaklogin-page.png)

Keycloak 登录表单">

在您的开发环境中设置 Keycloak 只需要几个步骤。看看 [Keycloak 快速入门](https://github.com/keycloak/keycloak-quickstarts) GitHub repo 中提供的[快速入门指南](https://github.com/keycloak/keycloak-quickstarts/tree/latest.)。

*Last updated: August 3, 2022*
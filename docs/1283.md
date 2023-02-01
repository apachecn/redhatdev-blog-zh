# JBoss EAP 7.1 的新增功能:凭证存储

> 原文：<https://developers.redhat.com/blog/2017/12/14/new-jboss-eap-7-1-credential-store>

在以前版本的 JBoss EAP 中，安全存储凭证和其他敏感字符串的主要方法是使用密码库。密码库使您不必在 JBoss EAP 配置文件中以纯文本形式保存密码和其他敏感字符串。

然而，密码保险库也有一些缺点。例如，每个 JBoss EAP 服务器只能使用一个密码库，并且密码库的所有管理都必须通过外部工具来完成。

JBoss EAP 7.1 中的`elytron`子系统新增了凭证存储特性。

您可以在 JBoss EAP 管理 CLI 中创建和管理多个凭据库，JBoss EAP 管理模型现在支持使用`credential-reference`属性引用凭据库中的值。您还可以使用 Elytron Client 为 Java 应用程序创建和使用凭证存储。

下面是一个快速演示，展示了如何使用 JBoss EAP 管理 CLI 创建和使用凭据库。

### **创建凭据库**

```
/subsystem=elytron/credential-store=my_store:add(location="cred_stores/my_store.jceks", relative-to=jboss.server.data.dir,  credential-reference={clear-text=supersecretstorepassword},create=true)
```

### **向凭据库添加凭据或敏感字符串**

```
/subsystem=elytron/credential-store=my_store:add-alias(alias=my_db_password, secret-value="speci@l_db_pa$$_01")
```

### **使用 JBoss EAP 配置中存储的凭证**

以下示例使用之前添加的凭据作为新 JBoss EAP 数据源的密码。

```
data-source add --name=my_DS --jndi-name=java:/my_DS --driver-name=h2 --connection-url=jdbc:h2:mem:test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE --user-name=db_user --credential-reference={store=my_store, alias=my_db_password}
```

### **在 EJB 应用程序中使用凭证商店** 

EJB 和其他客户机可以使用 [Elytron 客户机在 JBoss EAP 服务器之外创建、修改和访问凭证存储库](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/how_to_configure_server_security/#cred_store_elytron_client)。

有关在 JBoss EAP 7.1 中使用凭据存储的更多信息，包括如何将现有密码保管库转换为凭据存储，请参见 JBoss EAP 7.1 [*如何配置服务器安全性*](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/how_to_configure_server_security/#credential_store) 指南。

*Last updated: December 13, 2017*
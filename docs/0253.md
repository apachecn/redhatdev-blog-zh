# 如何在身份代理期间限制 Keycloak 中的用户身份验证

> 原文：<https://developers.redhat.com/blog/2020/12/30/how-to-restrict-users-being-authenticated-in-keycloak-during-identity-brokering>

根据设计，如果用户通过任何第三方身份提供商(例如，谷歌、脸书或 Okta)进行身份验证，Keycloak 会将所有用户导入其本地数据库。但是，如果通过第三方身份提供者认证的用户必须被限制——或者只允许有限的访问——使用 Keycloak 联合的应用程序，该怎么办呢？这是你怎么做的。

您首先开发一个自定义的认证器，如果第三方认证的用户超出已知列表，它将禁用该用户，并将该认证器放在`First Broker Login`认证流中。

**注**:本文假设你熟悉 Keycloak 和 Maven。 [Keycloak](https://www.keycloak.org/) 是一款开源的身份和访问管理(IAM)工具，是[红帽的单点登录(SSO)工具](https://developers.redhat.com/blog/2019/02/07/red-hat-single-sign-on-give-it-a-try-for-no-cost/)的上游项目。许多开发人员在生产环境中使用 Keycloak 或 Red Hat 的 SSO 工具来实现企业安全性。

## 使用 Keycloak 创建自定义验证器

Keycloak 提供了一个[认证服务提供者接口](https://www.keycloak.org/docs/latest/server_development/#_auth_spi) (SPI)，我们将使用它来编写一个新的定制认证器。正如 Keycloak 文档中的[所描述的，当我们打包自定义验证器时，我们必须做以下事情:](https://www.keycloak.org/docs/latest/server_development/#packaging-classes-and-deployment)

*   将整个实现打包到一个 JAR 文件中。
*   确保 JAR 包含一个名为`org.keycloak.authentication.AuthenticatorFactory`的文件。
*   在`META-INF/services/`目录中找到`org.keycloak.authentication.AuthenticatorFactory`文件。
*   确保它列出了每个`AuthenticatorFactory`实现的全限定类名。

## EnableIfRequire 类

首先，我们将创建两个类。第一个是`EnableIfRequire.java`，如果它存在于列表中，则允许用户创建帖子:

```
package com.sid.keycloakauthenticator;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;
import java.util.logging.Level;
import java.util.logging.Logger;
import org.keycloak.authentication.AuthenticationFlowContext;
import org.keycloak.authentication.authenticators.broker.IdpCreateUserIfUniqueAuthenticator;
import org.keycloak.authentication.authenticators.broker.util.SerializedBrokeredIdentityContext;
import org.keycloak.broker.provider.BrokeredIdentityContext;
import org.keycloak.models.UserModel;

/**
 *
 * @author sidd
 */
public class EnableIfRequire extends IdpCreateUserIfUniqueAuthenticator {
    //private final List users = Arrays.asList("siddhartha.de@mail.com","sidde3");
    private static List users = new ArrayList();

    static{
        File file = new File(System.getProperty("userlist")); //userlist is the reference of file which will hold the list of users
        if (file.exists()) {
            try {
                Scanner sc = new Scanner(file);
                sc.useDelimiter(",");
                while (sc.hasNext()) {
                    users.add(sc.next());
                }
            }catch (FileNotFoundException ex) {
                Logger.getLogger(CreateIfRequire.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }

    @Override
    protected void userRegisteredSuccess(AuthenticationFlowContext context, UserModel registeredUser, SerializedBrokeredIdentityContext serializedCtx, BrokeredIdentityContext brokerContext) {
        System.out.println(registeredUser.getUsername()+" User is successfully registered...");
        if(!users.contains(registeredUser.getUsername())){
            registeredUser.setEnabled(false);              //Disable the user if not there in list
        }

    }
}
```

## EnableIfRequireFactory 类

接下来，我们创建`EnableIfRequireFactory.java`，它实例化了验证者:

```
package com.sid.keycloakauthenticator;

import java.util.ArrayList;
import java.util.List;
import org.keycloak.Config;
import org.keycloak.authentication.Authenticator;
import org.keycloak.authentication.authenticators.broker.IdpCreateUserIfUniqueAuthenticatorFactory;
import org.keycloak.models.KeycloakSession;
import org.keycloak.provider.ProviderConfigProperty;

/*
 * @author sid
 */
public class EnableIfRequireFactory extends IdpCreateUserIfUniqueAuthenticatorFactory {

    public static final String PROVIDER_ID = "idp-enable-user-if-require";
    static CreateIfRequire SINGLETON = new CreateIfRequire();

    public static final String REQUIRE_PASSWORD_UPDATE_AFTER_REGISTRATION = "require.password.update.after.registration";

    @Override
    public Authenticator create(KeycloakSession session) {
        return SINGLETON;
    }

    @Override
    public void init(Config.Scope config) {

    }

    @Override
    public String getId() {
        return PROVIDER_ID;
    }

    @Override
    public String getDisplayType() {
        return "Enable User When Require";
    }

    @Override
    public String getHelpText() {
        return "Enable user when require";
    }

    private static final List configProperties = new ArrayList();

    static {
        ProviderConfigProperty property;
        property = new ProviderConfigProperty();
        property.setName(REQUIRE_PASSWORD_UPDATE_AFTER_REGISTRATION);
        property.setLabel("Require Password Update");
        property.setType(ProviderConfigProperty.BOOLEAN_TYPE);
        property.setHelpText("You are required to update password when user will be created");
        configProperties.add(property);
    }

    @Override
    public List getConfigProperties() {
        return configProperties;
    }
}

```

## 组织和编译 Keycloak 自定义验证器

在这一节中，我们将使用 Maven 来组织移动认证项目并编译我们的两个新类。

### 设置项目

执行以下命令，使用 Maven 创建一个项目:

```
mvn archetype:generate -DgroupId=com.sid.keycloakauthenticator -DartifactId=keycloak-authenticator -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

```

将我们刚刚创建的两个类放在`src/main/java/com/sid/keycloakauthenticator`路径中。

现在，在`src/main/resources/META-INF/services`创建一个名为`org.keycloak.authentication.AuthenticatorFactory`的文件。为新的`AuthenticationFactory` : `com.sid.keycloakauthenticator.EnableIfRequireFactory`增加一个条目。

### 解决项目依赖关系

Keycloak 认证模块是一个私有的 SPI，所以您需要使用`MANIFEST.MF`来解析依赖关系。在`src/main/resources/META-INF`行的`MANIFEST.MF`中输入以下内容:

```
Dependencies: org.keycloak.keycloak-server-spi-private, org.keycloak.keycloak-services, org.keycloak.keycloak-core, org.keycloak.keycloak-server-spi

```

您现在可以编辑 Maven `pom.xml`来添加以下依赖项:

```
        <dependency>
	   <groupId>org.keycloak</groupId>
	   <artifactId>keycloak-core</artifactId>
	   <version>4.8.3.Final</version>
	   <scope>provided</scope>
	</dependency>
	<dependency>
	   <groupId>org.keycloak</groupId>
	   <artifactId>keycloak-server-spi</artifactId>
	   <version>4.8.3.Final</version>
	   <scope>provided</scope>
	</dependency>
	<dependency>
	   <groupId>org.keycloak</groupId>
	   <artifactId>keycloak-server-spi-private</artifactId>
	   <version>4.8.3.Final</version>
	   <scope>provided</scope>
	</dependency>
	<dependency>
	   <groupId>org.jboss.logging</groupId>
	   <artifactId>jboss-logging</artifactId>
	   <version>3.4.0.Final</version>
	   <scope>provided</scope>
	</dependency>
	<dependency>
	   <groupId>org.keycloak</groupId>
	   <artifactId>keycloak-services</artifactId>
	   <version>4.8.3.Final</version>
	   <scope>provided</scope>
	</dependency>

```

### 构建和部署项目

执行以下命令来构建项目:

```
mvn clean install

```

该命令在`keycloak-authenticator-1.0-SNAPSHOT.jar`目标文件夹中生成输出。Keycloak 与 [WildFly](https://www.wildfly.org/) 捆绑在一起，因此您可以使用`jboss-cli`接口和以下命令来部署 JAR:

```
deploy /path/to/keycloak-authenticator-1.0-SNAPSHOT.jar

```

### 配置自定义身份验证流

成功部署认证器 JAR 之后，您将配置认证流。以下是如何在 Keycloak 中配置自定义流:

1.  登录 Keycloak 管理控制台，选择您想要配置自定义移动验证器的领域，然后在左侧面板中点击**验证**。
2.  在**流程**选项卡中，从下拉列表中选择**第一个经纪人登录**。
3.  点击**复制**按钮，给流程命名；例如，CustomBrokerFlow。
4.  点击**添加执行**，在提供者下拉列表中选择**需要时启用用户**
5.  将执行者放在**之后，如果唯一则创建用户**
6.  现在，该认证流可以用于相关身份提供者的第一次登录流。

*Last updated: December 29, 2020*
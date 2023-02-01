# 在 Keycloak 中使用手机号码进行用户身份验证

> 原文：<https://developers.redhat.com/blog/2020/10/23/use-mobile-numbers-for-user-authentication-in-keycloak>

我最近参与了一个项目，该项目要求使用手机号码进行用户认证，而不是传统的用户名和密码。几乎每个人都有一个独一无二的手机号码，所以这个要求是合理的。我们的身份验证工具是 [Keycloak](https://developers.redhat.com/blog/2020/08/07/a-deep-dive-into-keycloak/) ，它不附带基于移动设备的身份验证选项。相反，我的团队开发了一个定制的身份验证执行器来满足需求。

在本文中，我将向您展示如何使用 Keycloak 的身份验证服务提供者接口(SPI)来编写一个定制的`MobileAuthenticator`类，然后用一个`AuthenticationFactory`实例化它。我还将向您展示如何使用 Maven 打包和编译移动认证项目，以及如何为 Keycloak 创建定制的移动认证流。

**注**:本文假设你熟悉 Keycloak、Maven、[红帽 JBoss 企业应用平台](https://developers.redhat.com/products/eap/overview)。 [Keycloak](https://www.keycloak.org/) 是一款开源的身份和访问管理(IAM)工具，是[红帽单点登录](https://developers.redhat.com/blog/2019/02/07/red-hat-single-sign-on-give-it-a-try-for-no-cost/)(红帽单点登录)的上游项目。许多开发人员在生产环境中使用 Keycloak 或 Red Hat SSO 来实现企业安全性。

## 使用 Keycloak 创建自定义验证器

Keycloak 提供了一个[认证服务提供者接口](https://www.keycloak.org/docs/latest/server_development/#_auth_spi) (SPI)，我们将使用它来编写一个新的自定义认证器。正如 Keycloak 文档中的[所描述的，当我们打包自定义验证器时，我们必须做以下事情:](https://www.keycloak.org/docs/latest/server_development/#packaging-classes-and-deployment)

*   将整个实现打包到一个 JAR 文件中。
*   确保 JAR 包含一个名为`org.keycloak.authentication.AuthenticatorFactory`的文件。
*   在`META-INF/services/`目录中找到`org.keycloak.authentication.AuthenticatorFactory`文件。
*   确保它列出了每个`AuthenticatorFactory`实现的全限定类名。

## MobileAuthenticator 类

首先，我们将创建两个类。第一个是`MobileAuthenticator.java`，它执行认证:

```
package com.sid.keycloakauthenticator;
import java.util.List;
import javax.ws.rs.core.MultivaluedMap;
import org.keycloak.authentication.AuthenticationFlowContext;
import org.keycloak.authentication.Authenticator;
import org.keycloak.authentication.authenticators.browser.UsernamePasswordForm;
import org.keycloak.events.Errors;
import org.keycloak.services.managers.AuthenticationManager;

import javax.ws.rs.core.Response;
import org.keycloak.authentication.AuthenticationFlowError;
import org.keycloak.authentication.authenticators.browser.AbstractUsernameFormAuthenticator;
import org.keycloak.events.Details;
import org.keycloak.models.ModelDuplicateException;
import org.keycloak.models.UserModel;
import org.keycloak.services.messages.Messages;

/**
 * @author sid
 **/
public class MobileAuthenticator extends UsernamePasswordForm implements Authenticator {

   @Override
   public boolean validateUserAndPassword(AuthenticationFlowContext context, MultivaluedMap inputData) {
	String username = inputData.getFirst(AuthenticationManager.FORM_USERNAME);
	if (username == null) {
	   context.getEvent().error(Errors.USER_NOT_FOUND);
	   Response challengeResponse = challenge(context, Messages.INVALID_USER);
	   context.failureChallenge(AuthenticationFlowError.INVALID_USER, challengeResponse);
	   return false;
	}

	// remove leading and trailing whitespace
	username = username.trim();
	context.getEvent().detail(Details.USERNAME, username);
	context.getAuthenticationSession().setAuthNote(AbstractUsernameFormAuthenticator.ATTEMPTED_USERNAME, username);
	UserModel user = null;
	try {
	   List users = context.getSession().users().searchForUserByUserAttribute("mobile", username, context.getRealm());
	   System.out.println(users.get(0).getUsername());
	   if (users != null && users.size() == 1) {
		user = users.get(0);
	   }
	} catch (ModelDuplicateException mde) {
	   if (mde.getDuplicateFieldName() != null && mde.getDuplicateFieldName().equals(UserModel.EMAIL)) {
		setDuplicateUserChallenge(context, Errors.EMAIL_IN_USE, Messages.EMAIL_EXISTS, AuthenticationFlowError.INVALID_USER);
	   } else {
		setDuplicateUserChallenge(context, Errors.USERNAME_IN_USE, Messages.USERNAME_EXISTS, AuthenticationFlowError.INVALID_USER);
	   }
	   return false;
	}

	if (invalidUser(context, user)) {
	   return false;
	}

	if (!validatePassword(context, user, inputData)) {
	   return false;
	}

	if (!enabledUser(context, user)) {
	   return false;
	}

	String rememberMe = inputData.getFirst("rememberMe");
	boolean remember = rememberMe != null && rememberMe.equalsIgnoreCase("on");
	if (remember) {
	   context.getAuthenticationSession().setAuthNote(Details.REMEMBER_ME, "true");
	   context.getEvent().detail(Details.REMEMBER_ME, "true");
	} else {
	   context.getAuthenticationSession().removeAuthNote(Details.REMEMBER_ME);
	}
	context.setUser(user);

	return true;
   }
}
```

## MobileAuthenticationFactory 类

接下来，我们创建`MobileAuthenticationFactory.java`，它实例化了验证者:

```
package com.sid.keycloakauthenticator;

import org.keycloak.Config;
import org.keycloak.authentication.Authenticator;
import org.keycloak.authentication.authenticators.browser.UsernamePasswordFormFactory;
import org.keycloak.models.KeycloakSession;

/**
 * @author sid
 **/
public class MobileAuthenticationFactory extends UsernamePasswordFormFactory {

   public static final String PROVIDER_ID = "mobile-authenticator";
   public static final MobileAuthenticator SINGLETON = new MobileAuthenticator();

   @Override
   public Authenticator create(KeycloakSession session) {
	return SINGLETON;
   }

   @Override
   public void init(Config.Scope scope) {
   }

   @Override
   public String getId() {
	return PROVIDER_ID;
   }

   @Override
   public String getDisplayType() {
	return "Mobile Based User Form";
   }

   @Override
   public String getHelpText() {
	return "Validates a mobile and password from login form.";
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

现在，在`src/main/resources/META-INF/services`创建一个名为`org.keycloak.authentication.AuthenticatorFactory`的文件。为新的`AuthenticationFactory` : `com.sid.keycloakauthenticator.MobileAuthenticationFactory`增加一个条目。

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

1.  登录 Keycloak 管理控制台，选择要配置自定义移动认证器的领域，点击左侧面板中的**认证**
2.  在**流程**选项卡中，从下拉列表中选择**浏览器**。
3.  点击**复制**按钮，给流程命名；比如 MobileFlow。
4.  在 **MobileFlow Forms** 下，点击**动作**超链接添加执行。
5.  通过从提供商列表中选择**基于移动设备的用户表单**来保存流程。
6.  删除**用户名密码表**和 **OTP 表**。

## 结论

这就是使用 Keycloak 设置基于移动设备的身份验证的全部内容。请注意，为了成功进行身份验证，您必须确保每个用户在其属性中都有一个唯一的手机号码。

*Last updated: April 7, 2022*
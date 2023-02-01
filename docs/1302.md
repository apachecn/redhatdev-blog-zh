# 如何设置 3scale OpenID Connect (OIDC)与 RH SSO 的集成

> 原文：<https://developers.redhat.com/blog/2017/11/21/setup-3scale-openid-connect-oidc-integration-rh-sso>

本分步指南是 Red Hat 3scale API 管理新 2.1 版本[发布](https://developers.redhat.com/blog/2017/10/26/3scale-api-management-simplifies-openid-connect-integration/)的后续。正如你们中的许多人所知，这个新版本简化了 APIcast gateway 和 Red Hat Single Sign-On 之间的集成，通过 OpenID Connect (OIDC)进行 API 认证。因此，现在除了 API 密钥、App 密钥对和 OAuth 之外，您还可以选择 OpenID Connect 作为您的身份验证机制。此外，内部版本添加了一个新组件，该组件同步 Red Hat 单点登录域上的客户端创建。

## 介绍

像大多数这种类型的指南一样，本指南仅用于本地开发或演示新特性。这绝不是为了在生产环境中使用，因为它可能会绕过安全性和/或高可用性建议。

首先，您需要一个正在运行的 3 倍规模本地实例。我建议使用 openshift CDK 来设置一个可以在您的笔记本电脑或虚拟机上运行的本地环境。如果您没有运行环境，可以按照我的[HOW-TO-setup 3 scale on-premise](https://developers.redhat.com/blog/2017/05/22/how-to-setup-a-3scale-amp-on-premise-all-in-one-install/)指南从头开始设置一个。如果您在同一个 minishift 实例中部署 Red Hat Single Sign-On (RH-SSO ),请记住增加资源的数量。第一步允许您从头开始设置一个 RH-SSO 实例。如果您已经有一个正在运行的实例，请跳到第 2 节——配置 RH-SSO。

## 设置 Red Hat 单点登录

如果您没有正在运行的 Red Hat 单点登录实例，或者只是想为这个集成设置一个额外的实例，请遵循下面的步骤。

1.  使用**系统登录正在运行的实例:管理员**用户:

    ```
    $ oc login -u system:admin --insecure-skip-tls-verify=true <your-master-url>
    ```

2.  如果还没有，添加 JBoss 图像流:

    ```
    $ oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json -n openshift
    ```

3.  添加 RH-SSO 模板:

    ```
    $ for i in {https,mysql,mysql-persistent,postgresql,postgresql-persistent}; do oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/master/sso/sso71-$i.json -n openshift; done
    ```

4.  创建一个名为 rh-sso 的新项目:

    ```
    $ oc new-project rh-sso
    ```

5.  为 TLS 安全路由

    ```
    $ oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/ose-v1.4.7/secrets/sso-app-secret.json -n rh-sso
    ```

    创建*服务账号*和*秘密*
6.  将查看角色添加到服务帐户 sso-service-account:

    ```
    $ oc policy add-role-to-user view system:serviceaccount:rh-sso:sso-service-account
    ```

7.  在项目中创建一个新的 app 处理持久化 mysql 模板:

    ```
    $ oc new-app sso71-mysql-persistent -p HTTPS_NAME=jboss -p HTTPS_PASSWORD=mykeystorepass -p SSO_ADMIN_USERNAME=keyadmin -p SSO_ADMIN_PASSWORD=keypassword
    ```

8.  While most of the time, there are no problems with this version, if you notice in the web console that your SSO deployment does not start, edit the deployment configuration to change the image stream with the version available at the Red Hat container catalog.![Red Hat Container Catalog](img/24dc4ad9556997ea9716836415f2128e.png)

    ![sso Deployment Configuration](img/41374274f61f3a38bde2fc3facd24d3f.png)

9.  从您的浏览器访问安全路由:https://secure-sso-rh-sso。 <your-cdk-ip>.nip.io/auth/admin/</your-cdk-ip>
10.  Accept the self-signed certificate.

    ![self-signed certificate](img/20ebd6c7cea7ab77f121a9afd7e846dd.png)

11.  Log in to the console using username: **ssoUser** and password: ****ssoPassword. ****

    ![login](img/a274b5fbb1beed5cfcbf479845f65a37.png)

12.  Done! You now have a running instance of Red Hat Single Sign-On.

    ![sso realm](img/95c96e6bc7b4b45ddb409a13c60c7c11.png)

## 为 OpenId 连接配置 RH-SSO

现在您已经有了一个正在运行的 RH-SSO 实例，我们将需要添加一些配置步骤来允许 3scale 同步。

1.  Create a new realm named 3scale-sso.

    ![add realm](img/fd55ab3416946ba71b2d072d4e51f0fd.png)

    ![3scale-sso realm](img/ca3d20578b7ecf152160dd025e5daad2.png)

2.  Disable *Require SSL* for the realm (to simplify the Zync connection to RH-SSO self-signed certificate) under the **Login** tab.

    ![disable ssl](img/e49d9f8e874f3c47ca204108700e708b.png)

3.  签出。
4.  Login to the unsecured RH-SSO web console at http://sso-rh-sso.nip.io/auth/admin/ to validate it's now working without SSL.

    ![unsecured login](img/251824964612ad6b1123adef57261fa8.png)

5.  Click on the **Clients** menu on the left side and click the *Create* button.

    ![clients](img/b1564441edbaec248f0f24616af59ed3.png)

6.  Type **3scale-admin** as the *Client ID*, select **openid-connect** as the *Client Protocol* and click on the **Save** button.

    ![add client](img/1d22cab78d13345266f9c1d912ff4884.png)

7.  As this will be the service account used by 3scale to perform client synchronization, in the client settings select **confidential** as *Access Type*, turn OFF **Standard Flow Enabled** and **Direct Access Grants Enabled** and turn ON **Service Accounts Enabled**. Finally, click the **Save** button.

    ![service account settings](img/8cdb4c208e66730a7fab60f72b14b7ca.png)

8.  If the page did not refresh automatically, refresh it. This will enable the **Service Account Roles** tab for the client. Click on it.

    ![service account roles tab](img/d23b8afe70403edf79015aa527cc35d8.png)

9.  Select **realm-management** from the *Client Roles*.

    ![realm-management](img/4248f4c94f7277cfd82d145ec3dbf84e.png)

10.  Add **manage-clients** to the account.

    ![manage-clients](img/4a931e5c1d1991346468facb146981c1.png)

11.  Finally, click on the **Credentials** tab and take notice of the **Secret**. Write it down as you will use it to configure 3scale.

    ![client credentials](img/5102d6f29386163705b186da47924c94.png)

12.  向领域添加用户。
    1.  Click on the **Users** menu on the left side of the screen and click the **Add user** button.

        ![add user](img/25a818e57485c1a5c553bac37d17ce38.png)

    2.  Type **apiUser** as the *Username*. Click on the **Save** button.

        ![username](img/1d9bcae830b6aaaab14ec67c23709a3c.png)

    3.  Click on the **Credentials** tab to reset the password. Type **apiPassword** as the *New Password* and *Password Confirmation*. Turn *OFF* the **Temporary** to avoid the password reset at the next login.

        ![user credentials](img/e7dbe7ec9658d5626ca909c748fcf625.png)

    4.  Click on the **Change password** button in the pop-up dialog.

        ![change password dialog](img/c2f4485110b2f2a9387211555e118d9a.png)

    5.  搞定了。现在您有一个用户来测试您的集成。

## 配置 3 级集成

1.  登录您的 3scale 管理门户网站。
2.  Select the service you want to enable OpenId Connect integration with RH-SSO. Click on the **APIs** tab, select the *Service* and click on the **Integration** link. We are using the default *API*.

    ![service integration](img/37d747219100968374771ba0e5cc231e.png)

3.  在该页面中，点击 **编辑整合设置** 。

    ![edit integration settings](img/89f8e8e195927e8125ab1dcb2111ba12.png)

4.  Under the *Authentication* deployment options, select **OpenID Connect**. Click on the **Update Service** button.

    ![openid connect authentication](img/17c79b19a554adc489a072f19f8a4634.png)

5.  Back in the service integration, click on the **edit APIcast configuration**.

    ![edit apicast configuration](img/702515acc17f8365dafca20e53b262fc.png)

6.  Expand the authentication options by clicking *Authentication Settings*.

    ![openid connect issuer](img/9f266d3dbfe43d4ab839495cf363b265.png)

7.  在 **OpenID Connect Issuer** 字段中，输入您之前提到的带有 RH-SSO 服务器 URL 的客户端凭证。

    ```
    http://3scale-admin:<CLIENT-SECRET>@sso-rh-sso.<YOUR-CDK-IP>.nip.io/auth/realms/3scale-sso
    ```

8.  最后，点击**更新暂存环境**按钮。
9.  (可选)点击**提升至生产**按钮，提升至生产。
10.  创建一个新的应用程序，以便 3scale 可以将其与 RH-SSO 同步:
    1.  Go to the *Developers* tab and click on **Developer**.

        ![developers](img/0656671ffacbb4a8227a263e89a078fe.png)

    2.  Click on the **Applications** link.

        ![applications](img/8b87c28001fa70b9b84fd3aaa7f27559.png)

    3.  Click on **Create Application** link.

        ![create application](img/ae8192502acdcd79ee986e63e41087f3.png)

    4.  Select an application plan from the service you are securing. In our case is the Echo API. Type **Secure App** in the *Name* field. Type **OpenID Connect Secured Application** in the *Description* box. Finally, click on the **Create Application** button.

        ![application details](img/410a793659d1614e494870911b81c35f.png)

    5.  Note the API Credentials. Write them down as you will need the **Client ID** and the **Client Secret** to test your integration. Click on the **Change** link from *Redirect URL*.

        ![api credentials](img/4aa36a9bc3695fb10a68c8f2c3878bbc.png)

    6.  We will use the Postman to test our integration so we will fill in the callback information with a fixed link. Type in **https://openidconnect.net/callback** in the *Redirect URL* field. Click on the **Update** button.

        ![](img/fd61f86796c3aa65f301d1fa7ff3ab74.png)

    7.  恭喜你！现在您有了一个应用程序来测试您的 OpenId Connect 集成。
11.  Login to the RH-SSO console if you are not there already and click on the Clients menu. Now you can check that 3scale zync component creates a new Client in RH-SSO. This new Client has the same ID as the *Client ID* from the 3scale admin portal.![app client id](img/97b585e2d8e6f1a394e69cc930ded73d.png)

    ![rhsso client id](img/b7a71e9f5fdb509804416aaac10dd63b.png)

## 测试集成

在检查我们的应用程序客户机已经在 RH-SSO 中创建之后，我们可以继续使用 Postman 测试集成。

1.  Open Postman and click on create a new **Request**.

    ![postname new request](img/84eb612077a45cf9c30d4157d7436147.png)

2.  Type **Secure API** in the *Request Name* field. Click on **+ Create Collection** button.

    ![create collection](img/2446ec455b53f4606dff520e72cb6aa8.png)

3.  Type **Secure API Collection** in the editable field. Click on the checkmark.

    ![collectio name](img/4884de4e845dd5f8bc0a5de382bbe99b.png)

4.  Click on **Save to Secure API Collection** button.

    ![save request](img/bd8bf1c230cea69f8c24527df612e2b9.png)

5.  Select **OAuth 2.0** from the *Authorization* *TYPE* combobox.

    ![oauth](img/e3a91171689a0f0635a731168b95fcb2.png)

6.  点击**获取新的访问令牌**按钮。
7.  使用以下信息填写设置配置:
    *   回拨网址:**https://openidconnect.net/callback**
    *   Auth URL: **http://sso-rh-sso。<YOUR-open shift-IP>. nip . io/auth/realms/3 scale-SSO/protocol/OpenID-connect/auth**
    *   访问令牌 URL: **http://sso-rh-sso。<YOUR-open shift-IP>. nip . io/auth/realms/3 scale-SSO/protocol/OpenID-connect/token**
    *   客户 ID: **<你的客户 ID >**
    *   Client Secret: ****<YOUR-CLIENT-SECRET> ****

        ![auth config](img/e333b2e6e3b8bd25c0c619ccc3b71bc3.png)

8.  Click on the **Request Token** button. You will be redirected to the RH-SSO login page. Log in with your user credentials created in the previous steps: **apiUser/apiPassword**. 

    ![rhsso login](img/3d3b441a53483ab0aa3bd8f7121078be.png)

9.  点击**登录**按钮。
10.  If successful, you will see a page with the generated token. Scroll to the bottom of the page and click on the **Use Token** button.

    ![use token](img/292bc9065356e2ff49161a27a4122c58.png)

11.  Enter the request URL in the GET field for the staging endpoint for your secured API:![staging url](img/830a3a3d3cdb490e2260eb86dc5b77bf.png)

    ![secured api url](img/03dd3b0dd3e7797a0395c5c893f1914c.png)

12.  Click on the blue **Send** button to execute the request using the selected token. You will see the return from the Echo API.

    ![request response](img/5c8312b428060fd3688241e3f4fab888.png)

13.  如果您的令牌已过期或不正确，您将收到一个授权错误。
14.  恭喜您，您的 API 现已通过 3scale API 管理 OpenID Connect 集成得到保护。

## 后续步骤

现在，您可以使用 Red Hat 单点登录的三分支身份验证来保护您的 API，您可以利用您组织的当前资产，如当前的 LDAP 身份，甚至可以使用其他 IdP 服务联合身份验证。

有关单点登录的更多信息，可以查看它的[页面](https://access.redhat.com/products/red-hat-single-sign-on)。

关于 3scale API 管理的见解，可以回顾以下[链接](https://access.redhat.com/products/red-hat-3scale/)。

* * *

**如果你知道 Linux 的基本命令，那么下载** [**高级 Linux 命令备忘单**](https://developers.redhat.com/cheat-sheets/advanced-linux-commands/) **，这个备忘单可以帮助你把你的技能提升到一个新的水平。**

*Last updated: September 3, 2019*
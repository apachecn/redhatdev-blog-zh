# 使用模板在 Red Hat OpenShift 上部署 Red Hat 的单点登录技术

> 原文：<https://developers.redhat.com/articles/2021/05/19/deploy-red-hats-single-sign-technology-red-hat-openshift-using-templates>

本文的目标是帮助开发者理解和使用 [Red Hat OpenShift 图像流和应用程序模板](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html/red_hat_single_sign-on_for_openshift_on_openjdk/get_started#image-streams-applications-templates)在 [OpenShift](/products/openshift/overview) 上部署 Red Hat 的单点登录(SSO)技术。另见我之前关于使用 SSO 和 OpenShift 的文章:[将红帽的单点登录技术 7.4 与红帽 OpenShift](/blog/2021/03/25/integrate-red-hats-single-sign-on-technology-7-4-with-red-hat-openshift/) 集成。

## OpenShift 部署的 SSO 模板

使用以下命令获取 OpenShift 的 SSO 部署模板列表:

```
$ oc get templates -n openshift -o name | grep -o 'sso74.\+'
sso74-https
sso74-postgresql
sso74-postgresql-persistent
sso74-x509-https
sso74-x509-postgresql-persistent
```

### 为您想要的行为选择模板

您可以使用模板来定义如何在 OpenShift 上部署 SSO。每个模板都提供了不同的部署选项:

*   TLS 终止:
    *   重新加密
    *   传递
*   数据库后端类型:
    *   内存中的 H2 数据库
    *   PostgreSQL 数据库
*   存储:
    *   短暂的
    *   坚持的

您可以使用模板的名称模式来选择提供您需要的行为的模板。例如，模板名称中的`ocp4-x509`表示模板使用重新加密传输层安全性(TLS)终止。如果模板名包含`postgresql`，那么该模板的数据库后端就是 PostgreSQL。如果它包括`persistent`，那么您就知道该模板跨数据库或 SSO pod 重启保存 SSO 数据。

### SSO 模板功能

下面列出了 SSO 模板及其提供的功能:

单点登录模板:`sso74-https`:

*   直通 TLS 终端
*   H2 内存数据库
*   短暂储存

单点登录模板:`sso74-ocp4-x509-https`:

*   重新加密 TLS 终止
*   H2 内存数据库
*   短暂储存

单点登录模板:`sso74-ocp4-x509-postgresql-persistent`:

*   重新加密 TLS 终止
*   PostgreSQL 数据库
*   持久存储

单点登录模板:`sso74-postgresql`:

*   直通 TLS 终端
*   PostgreSQL 数据库
*   短暂储存

单点登录模板:`sso74-postgresql-persistent`:

*   直通 TLS 终端
*   PostgreSQL 数据库
*   持久存储

## SSO 模板中的 TLS 终止

名为模式`ocp4-x509`的单点登录模板使用重新加密 TLS 终止。所有其他模板都使用直通 TLS 端接。让我们考虑一下重新加密 TLS 终止和直通 TLS 终止之间的主要区别。

### 重新加密 TLS 终止

在重新加密 TLS 终止中，当您在 OpenShift 上部署单点登录时，会自动创建 SSO 密钥库和信任库。部署 TLS 非常简单，因为一切都是自动配置的。另一方面，您不能直接访问 SSO 密钥库和信任库。向 SSO 信任库添加新的外部证书颁发机构(CA)证书需要重建和重新部署新的 SSO 映像。

### 直通 TLS 终端

这种 TLS 终止方式非常灵活，更新 SSO 信任库也很容易。添加外部证书并不复杂。这种方法的缺点是必须手动配置，如这里的[所述](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html/red_hat_single_sign-on_for_openshift_on_openjdk/advanced_concepts#requirements_and_deploying_link_xl_href_introduction_introduction_xml_passthrough_templates_passthrough_tls_termination_link_red_hat_single_sign_on_templates)。

## 使用模板在 OpenShift 上部署 SSO

接下来，我们将使用模板在 OpenShift 上部署 SSO。

### 使用重新加密 TLS 终止模板在 OpenShift 上部署 SSO

下面是使用一个重新加密模板创建项目的命令。在本例中，我们选择 PostgreSQL 数据库和持久存储:

```
$ oc new-project sso-74-reencrypt
$ oc new-app --template=sso74-ocp4-x509-postgresql-persistent
```

请记住，使用重新加密 TLS 终止模板，将为您配置部署。接下来，我们将使用一个直通 TLS 终止模板部署 SSO。这些需要手动配置。

### 使用直通 TLS 终止模板在 OpenShift 上部署 SSO

在使用直通 TLS 终止模板部署 SSO 之前，我们必须手动创建 SSO 密钥库和信任库。密钥库和信任库作为 OpenShift 机密传递给 SSO pod。首先，我们将为单点登录服务器创建 HTTPS 和 JGroups 密钥库和信任库，然后我们将部署 SSO 模板。

#### 步骤 1:创建密钥库和信任库

首先创建 HTTPS 密钥库:

```
Generate a CA certificate. Pick and remember the password. Provide identical password, when signing the certificate sign request with the CA certificate below:

$ openssl req -new -newkey rsa:4096 -x509 -keyout xpaas.key -out xpaas.crt -days 365 -subj "/CN=xpaas-sso-demo.ca"
Generate a CA certificate for the HTTPS keystore. Provide mykeystorepass as the keystore password:

$ keytool -genkeypair -keyalg RSA -keysize 2048 -dnsureame "CN=secure-sso-sso-app-demo.openshift.example.com" -alias jboss -keystore keystore.jks
Generate a certificate sign request for the HTTPS keystore. Provide mykeystorepass as the keystore password:

$ keytool -certreq -keyalg rsa -alias jboss -keystore keystore.jks -file sso.csr
Sign the certificate sign request with the CA certificate. Provide the same password that was used to generate the CA certificate:

$ openssl x509 -req -CA xpaas.crt -CAkey xpaas.key -in sso.csr -out sso.crt -days 365 -CAcreateserial
Import the CA certificate into the HTTPS keystore. Provide mykeystorepass as the keystore password. Reply yes to Trust this certificate? [no]: question:

$ keytool -import -file xpaas.crt -alias xpaas.ca -keystore keystore.jks
Import the signed certificate sign request into the HTTPS keystore. Provide mykeystorepass as the keystore password:

$ keytool -import -file sso.crt -alias jboss -keystore keystore.jks
```

`keystore.jks`、`truststore.jks`和`jgroups.jceks`作为 OpenShift 秘密传递给 SSO pods。我们将这些机密链接到默认服务帐户，该帐户用于运行单点登录 pod:

```
$ oc create secret generic sso-app-secret --from-file=keystore.jks --from-file=jgroups.jceks --from-file=truststore.jks
secret/sso-app-secret created
$ oc secrets link default sso-app-secret
```

#### 步骤 2:部署 SSO 模板

现在，我们可以使用直通 TLS 终止模板在 OpenShift 上部署 SSO:

```
$ oc new-project sso-74-passthrough
$ oc new-app --template=sso74-postgresql-persistent
```

**注**:详细请参见[部署直通 TLS 终端单点登录模板的要求](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html/red_hat_single_sign-on_for_openshift_on_openjdk/advanced_concepts#requirements_and_deploying_link_xl_href_introduction_introduction_xml_passthrough_templates_passthrough_tls_termination_link_red_hat_single_sign_on_templates)。

## 向 SSO 信任库添加证书

如果 SSO 使用 HTTPS 或 LDAP 与外部服务器建立出站连接，我们可能需要向 SSO 信任库添加一个证书。当我们使用 X509 模板之一时(例如用于重新加密 TLS 终止)，SSO 密钥库和信任库是从 OpenShift 的内部 X509 证书服务自动生成的。为了添加新证书，我们需要构建新的 SSO 映像。

### 使用重新加密 TLS 终止模板添加证书

向 SSO 信任库添加证书包括以下步骤:

1.  从 SSO pod 中检索`java cacerts`文件。
2.  将`new certificate`添加到位于`/etc/pki/ca-trust/extracted/java/cacerts`的`java cacerts`文件中。(在部署期间，文件被自动复制到 SSO `/opt/eap/keystores/truststore.jks`中。)
3.  构建新的 SSO OpenShift 映像。
4.  部署新的 SSO 映像。

#### 步骤 1:从 SSO pod 中检索 java 证书

使用`rsync`命令从 SSO 运行窗格中检索 SSO `java cacerts`文件:

```
$ oc get pods

NAME READY STATUS RESTARTS AGE
<sso-pod-name> 1/1 Running 0 52s

$ oc rsync <sso-pod-name>:/etc/pki/ca-trust/extracted/java/cacerts .
```

#### 步骤 2:将新证书添加到 java cacerts 中

向 SSO 信任库添加新证书包括将这个新证书添加到`java cacerts`文件:

```
$ keytool -import -v -file <my-cert>.crt -alias my-cert -keystore ./cacerts -noprompt -storepass changeit
```

#### 步骤 3:构建新的 SSO OpenShift 映像

Dockerfile 描述了如何添加`java cacerts`文件来构建新的 SSO OpenShift 映像:

```
FROM rh-sso-7/sso74-openshift-rhel8:latest
COPY cacerts /etc/pki/ca-trust/extracted/java/cacerts
```

我们可以使用 Podman 命令来构建新的映像:

```
podman build -t docker-registry-default/project/name:tag .

podman build -t docker-registry-default/project/sso74:reencrypt-truststore-updated .
STEP 1: FROM rh-sso-7/sso74-openshift-rhel8:latest
STEP 2: COPY cacerts /etc/pki/ca-trust/extracted/java/cacerts
STEP 3: COMMIT docker-registry-default/project/sso74:reencrypt-truststore-updated
--> 356f138e3fd
356f138e3fdee0b7424eec5fec10f3b57ced6c896b829d55be1d60abc18280a3
```

新映像现在包含更新的 SSO 信任库。

#### 步骤 4:部署新的 SSO 映像

构建完成后，您将把您的新映像推送到 OpenShift 注册中心进行部署。当 SSO pod 重新启动时，将会上传新的映像。

### 使用直通 TLS 终止模板添加证书

使用此模板向 SSO 信任库添加新证书包括以下步骤:

1.  检索现有的`truststore.jks`文件。
2.  向`truststore.jks`添加新证书。
3.  删除对应于 SSO 信任库的机密。
4.  创建一个包含`truststore.jks`文件的新秘密。

#### 步骤 1:检索 SSO truststore.jks 文件

使用`rsync`命令从 OpenShift 中检索 SSO `truststore.jks`文件:

```
$ oc rsync <sso-pod-name>:/opt/eap/keystores/truststore.jks .
```

#### 步骤 2:向 truststore.jks 添加新证书

将这个新证书添加到 Java `cacerts`文件中:

```
$ keytool -import -v -file <my-cert>.crt -alias my-cert -keystore ./truststore.jks -noprompt -storepass changeit
```

#### 步骤 3:删除对应于 SSO 信任库的机密

更新 SSO 信任库包括更新 SSO pod 使用的密码。当密码的值被修改时，您必须删除原来的 pod 并创建一个新的 pod:

```
$ oc delete secret  sso-app-secret
```

#### 步骤 4:创建一个包含 truststore.jks 文件的新秘密

更新`truststore.jks`以包含您的新证书后，您必须重新创建密码`sso-app-secret`。更新的秘密包括`jgroups.jceks`、`truststore.jks`和`keystore.jks`:

```
$  oc create secret generic sso-app-secret --from-file=keystore.jks --from-file=jgroups.jceks --from-file=truststore.jks
secret/sso-app-secret created

$ oc secrets link default sso-app-secret
```

现在，您可以安全地将 SSO 与部署在 OpenShift 上的更新的信任库一起使用了。

## 结论

模板为在 Red Hat OpenShift 上部署 Red Hat 的单点登录技术提供了非常有趣的功能。作为最终用户，您可以使用其中一个重新加密 TLS 终止模板(自动配置 TLS)或传递终止 TLS 模板(手动配置 TLS)。正如您在本文中看到的，使用直通 TLS 终止模板时，向 SSO 信任库添加新证书要容易得多。您可以通过更新`trustsrore.jks`文件的秘密来动态地完成这项工作。如果您使用重新加密 TLS 终止模板，您将需要重建并重新部署原始 SSO 映像。

*Last updated: August 26, 2022*
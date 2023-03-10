# 在 Red Hat OpenShift 上集成 Red Hat 数据网格和 Red Hat 单点登录技术

> 原文：<https://developers.redhat.com/blog/2021/04/23/integrate-red-hat-data-grid-and-red-hats-single-sign-on-technology-on-red-hat-openshift>

使用[红帽数据网格](/products/datagrid/overview)作为[红帽单点登录技术](https://access.redhat.com/products/red-hat-single-sign-on)的外部缓存，使得数据网格能够独立于应用层存储数据。通过这种方式，数据网格提供了应用程序弹性、跨数据中心的故障转移以及减少的内存占用。

这种组合最常见的用例是[跨数据中心复制模式](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html-single/server_installation_and_configuration_guide/index#crossdc-mode)，其中 Red Hat 的单点登录(SSO)技术使用数据网格在数据中心之间复制数据。

要在 [xPaaS 环境](https://www.redhat.com/en/blog/welcome-to-the-world-of-xpaas)中跨站点备份数据，如 [Red Hat OpenShift](/products/openshift/overview) ，推荐的方法是[使用数据网格操作器](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.1/html-single/running_data_grid_on_openshift/index#backup_sites)部署数据网格。

本文提供了如何让 Red Hat Data Grid 8.1.1 与 Red Hat 的单点登录技术版本 7.4.5 一起工作的快速说明。该文章没有使用完整的跨数据中心设置；只有一个数据网格服务器和一个运行在 OpenShift 上的 SSO 客户端。我们将使用飞车手罗德协议进行通信，同时启用身份验证和安全套接字层(SSL)。

**注意**:数据网格可以用作特定于应用程序的数据的外部缓存容器。作为外部缓存，它允许数据层独立于应用程序进行扩展。它还允许位于不同域的不同集群访问来自同一个数据网格集群的数据。

## 设置环境

本文中描述的集成需要以下技术:

*   笔记本电脑上的`oc`客户端
*   红帽的单点登录技术 7.4.5
*   数据网格 8.1.1
*   OpenShift 4.6.x

要为集成创建项目，请运行:

```
$ oc new-project *project_name*

```

## 设置数据网格

使用数据网格操作器安装数据网格 8.1.1(参见 [*在 Red Hat OpenShift*](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.1/html/running_data_grid_on_openshift/installation#create_olm_subscription) 上安装数据网格操作器的说明)。

```
NAME                                  READY   STATUS    RESTARTS   AGE

infinispan-operator-88d585dd7-xc5xh   1/1     Running   0          58s

```

为群集创建一个 Infinispan 自定义资源(CR ),并为您需要的每个缓存创建一个缓存 CR。

操作员可以轻松地在各种配置中部署数据网格。一旦从 OpenShift [OperatorHub](https://docs.openshift.com/container-platform/4.5/operators/understanding/olm-understanding-operatorhub.html) 部署了操作符，它就会公开定制资源(称为 Infinispan 集群和 Infinispan 缓存)，Data Grid 使用这些资源在现有集群上提供缓存。

为了将自定义缓存定义与数据网格功能(如跨站点复制)一起使用，我们创建了数据网格服务节点集群。这些节点使用数据网格即服务创建一个 Infinispan 集群。

首先，通过进入**已安装的操作符— > Infinispan 集群— >创建**，从 OpenShift 控制台创建一个 Infinispan 集群。然后，在**服务**字段下选择**数据网格**服务类型。图 1 显示了这些选择。

[![The OpenShift console dialog to create an Infinispan cluster using Data Grid as a service.](img/f6334b8aba4965e0df3b2f159f4817fc.png "Screenshot from 2021-03-03 10-52-29")](/sites/default/files/blog/2021/03/Screenshot-from-2021-03-03-10-52-29-e1614749206488.png)

Figure 1: Create an Infinispan cluster using Data Grid as a service.

Infinispan CR 将如下所示:

```
$ oc get infinispan
NAME AGE
rhsso-infinispan 53s

```

您可以按如下方式获取窗格的状态:

```
$ oc get pods
NAME                                              READY     STATUS    RESTARTS          AGE
infinispan-operator-88d585dd7-xc5xh 1/1                    Running        0            6m43s
rhsso-infinispan-0                                 1/1     Running        0            100s
rhsso-infinispan-1                                 1/1     Running        0            29s

```

### 创建用于身份验证的基本身份验证密码

数据网格操作员必须通过数据网格服务集群的身份验证才能创建缓存。我们将在`basic-auth` secret 中添加凭证，这样数据网格操作员就可以在创建缓存时访问 Infinispan 集群。

您可以通过在 OpenShift 控制台的表格中填写详细信息来创建一个`basic-auth`密钥/值秘密。或者，您可以创建一个 YAML 文件，如 [*11.5.1 所述。添加凭证创建缓存*](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.1/html-single/running_data_grid_on_openshift/index#cache_auth-caches) 。

要从由密码生成的操作员处检索开发人员凭据，请运行:

```
oc get secret rhsso-infinispan-generated-secret \
-o jsonpath="{.data.identities\.yaml}" | base64 --decode

```

图 2 显示了创建密钥/值秘密的对话框。

[![Create a key/value secret in the OpenShift console.](img/ed9d27d3b26489dcadc84d8ec8c7a152.png "secret")](/sites/default/files/blog/2021/03/secret.png)

Figure 2: Create a key/value secret.

### 创建 Infinispan 缓存

使用操作符，[为您需要的每个缓存创建一个缓存 CR。您可以使用以下 CR 来创建工作缓存。同样，创建其他缓存，如`sessions`、`authenticationSessions`、`offlineSessions`、`clientSessions`、`offlineClientSessions`、`loginFailures`和`actionTokens`:](https://access.redhat.com/documentation/en-us/red_hat_data_grid/8.1/html/running_data_grid_on_openshift/caches#cache_xml-caches)

```
apiVersion: infinispan.org/v2alpha1
kind: Cache
metadata:
name: work
namespace: rhsso744dg81
spec:
adminAuth:
secretName: basic-auth
clusterName: eap-infinispan
name: work
template: >-
<infinispan><cache-container><replicated-cache name="work" mode="SYNC"
start="EAGER"><transaction mode="NONE" locking="PESSIMISTIC"/><locking
acquire-timeout="0" /></replicated-cache></cache-container></infinispan>

```

完成后，缓存状态应该如图 3 所示。

[![A list of Infinispan caches in the Data Grid Operator.](img/ee480d44ef87538d86be3947a0a2d5ce.png "cache")](/sites/default/files/blog/2021/03/cache-e1614999552205.png)

Figure 3: Infinispan caches in the Data Grid Operator.

### 创建路线

接下来，创建访问数据网格控制台的路由，如图 4 所示。

按照以下格式填写主机名:

```
*NAME_OF_THE_ROUTE*-*PROJECT_NAME*.apps.cndcluster9.ocp.gsslab.pnq2.redhat.com
```

例如:

```
dg-rhsso744dg81.apps.cndcluster9.ocp.gsslab.pnq2.redhat.com
```

[![The dialog to create the route.](img/ab9fa16a1d211afafee5e9bd22af56a0.png "route")](/sites/default/files/blog/2021/03/route.png)

Figure 4: Create the route for the Data Grid console.

### 建立飞车手罗德连接

最后，从`rhsso-infinispan-cert-secret`(由数据网格操作者生成的秘密)中检索`tls.crt`文件，以将其用于飞车手罗德连接:

```
$ oc get secret rhsso-infinispan-cert-secret \
> -o jsonpath='{.data.tls\.crt}' | base64 --decode > tls.crt

```

## 设置 SSO 客户端

要使用本节中的说明设置单点登录，请参考 SSO 服务器的[配置密钥库](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html-single/red_hat_single_sign-on_for_openshift_on_openjdk/index#Configuring-Keystores)和[配置密码](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html-single/red_hat_single_sign-on_for_openshift_on_openjdk/index#Configuring-Secrets)的文档。所需的任务是:

1.  创建 HTTPS 密钥库。
2.  为 JGroups 密钥库生成安全密钥。
3.  创建和链接秘密。

### 创建 HTTPS 密钥库

输入以下命令来生成 CA 证书。用下面的 CA 证书签署证书签名请求时，请提供相同的密码:

```
$ openssl req -new -newkey rsa:4096 -x509 -keyout xpaas.key -out xpaas.crt -days 365 -subj "/CN=xpaas-sso-demo.ca"
Generating a 4096 bit RSA private key
............................................................................................................................................................................................................................++
.....................................................................................................++
writing new private key to 'xpaas.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:

```

接下来，为 HTTPS 密钥库生成一个 CA 证书。提供密钥库的密码:

```
$ keytool -genkeypair -keyalg RSA -keysize 2048 -dname "CN=secure-sso-sso-app-demo.openshift.example.com" -alias jboss -keystore keystore.jks

Enter keystore password:
Re-enter new password:
Enter key password for <jboss>
(RETURN if same as keystore password):

```

输入以下命令为 HTTPS 密钥库生成证书签名请求:

```
$ keytool -certreq -keyalg rsa -alias jboss -keystore keystore.jks -file sso.csr
Enter keystore password:

#list the Generated files:
$ ls
tls.crt
xpaas.key
xpaas.crt
keystore.jks
sso.csr

```

签署证书-使用 CA 证书签署请求。提供您用来生成 CA 证书的相同密码:

```
$ openssl x509 -req -CA xpaas.crt -CAkey xpaas.key -in sso.csr -out sso.crt -days 365 -CAcreateserial
Signature ok
subject=/CN=secure-sso-sso-app-demo.openshift.example.com
Getting CA Private Key
Enter pass phrase for xpaas.key:

```

使用与上面相同的密钥库密码，将 CA 证书导入 HTTPS 密钥库。回复`yes`到`Trust this certificate? [no]:`:

```
 $ keytool -import -file xpaas.crt -alias xpaas.ca -keystore keystore.jks Enter keystore password: Trust this certificate? [no]: yes
```

使用相同的密钥库密码将已签名的证书签名请求导入 HTTPS 密钥库:

```
$ keytool -import -file sso.crt -alias jboss -keystore keystore.jks

Enter keystore password:
Certificate reply was installed in keystore

```

### 为 JGroups 密钥库生成安全密钥

输入以下命令以提供密钥库密码:

```
$ keytool -genseckey -alias secret-key -storetype JCEKS -keystore jgroups.jceks 

Enter keystore password: 
Re-enter new password: Enter key password for <secret-key> (RETURN if same as keystore password):
```

将 CA 证书导入新的 SSO 服务器信任库，并提供信任库密码。对`Trust this certificate? [no]:`问题回复`yes`:

```
$ keytool -import -file xpaas.crt -alias xpaas.ca -keystore truststore.jks

Enter keystore password:
Re-enter new password:  Trust this certificate? [no]: yes

```

### 创建并链接一个秘密

输入以下命令，为前面部分生成的 HTTPS 和 JGroups 密钥库以及 SSO 服务器信任库创建密钥:

```
$ oc create secret generic sso-app-secret --from-file=keystore.jks --from-file=jgroups.jceks --from-file=truststore.jks secret/sso-app-secret created
```

将这些机密链接到用于运行 SSO pods 的默认服务帐户:

```
$ oc secrets link default sso-app-secret

```

验证密码是否链接到服务帐户:

```
$ oc describe serviceaccounts default
Name: default
Namespace: rhsso744dg81
Labels: <none>
Annotations: <none>
Image pull secrets: default-dockercfg-gnvw9
Mountable secrets: default-token-pjmzp
default-dockercfg-gnvw9
sso-app-secret
Tokens: default-token-k4k6m
default-token-pjmzp
Events: <none>

```

## 部署 SSO 映像

我们将使用应用程序模板来部署单点登录映像。步骤如下:

1.  从 GitHub:

    ```
    $ git clone https://github.com/jboss-container-images/redhat-sso-7-openshift-image
    ```

    克隆模板
2.  更改到模板目录:

    ```
    $ cd redhat-sso-7-openshift-image/templates
    ```

3.  使用适当的参数将应用程序与模板一起部署:

    ```
    $ oc new-app --template=sso74-https -p HTTPS_SECRET="sso-app-secret" -p HTTPS_KEYSTORE="keystore.jks" -p HTTPS_NAME="jboss" -p HTTPS_PASSWORD="redhat" -p JGROUPS_ENCRYPT_SECRET="sso-app-secret" -p JGROUPS_ENCRYPT_KEYSTORE="jgroups.jceks" -p JGROUPS_ENCRYPT_NAME="secret-key" -p JGROUPS_ENCRYPT_PASSWORD="redhat" -p SSO_TRUSTSTORE="truststore.jks" -p SSO_TRUSTSTORE_PASSWORD="redhat" -p SSO_TRUSTSTORE_SECRET="sso-app-secret"
    ```

4.  设置数据网格时，转到从`rhsso-infinispan-cert-secret`(操作员生成的密码)中检索到`tls.crt`文件的目录。此证书将用于飞车手罗德连接:

    ```
    $ keytool -importcert -file tls.crt -keystore truststore.jks
    ```

5.  Create a secret with the truststore file, and add the volume and volume mount entries on the deployment configuration:

    ```
    $ oc create secret generic truststore-secret --from-file=truststore.jks
    secret/truststore-secret created

    $ oc set volume dc/sso --add --name=truststore-secret -m /etc/truststore -t secret --secret-name=truststore-secret --default-mode='0755'

    ```

    或者，您可以在 web 控制台上导航到秘密，点击**将秘密添加到工作负载**，并填写表格。在将秘密添加为卷并指定路径(`/etc/truststore`)之后，您将能够使用`/etc/truststore/truststore.jks`作为信任库。图 5 显示了向工作负载添加秘密的选项。

    [![Add a secret to the workload in the OpenShift console.](img/9918ba513c052a46ad3f624f0a4cf002.png "trust")](/sites/default/files/blog/2021/03/trust.png)

    Figure 5: Add a secret to the workload.

6.  验证音量:

    ```
    $ oc get dc
    NAME REVISION DESIRED CURRENT TRIGGERED BY
    sso 2 1 1 config,image(sso74-openshift-rhel8:7.4)
    $ oc set volume dc/sso --all
    sso
    secret/sso-app-secret as eap-keystore-volume
    mounted at /etc/eap-secret-volume
    secret/sso-app-secret as eap-jgroups-keystore-volume
    mounted at /etc/jgroups-encrypt-secret-volume
    secret/sso-app-secret as sso-truststore-volume
    mounted at /etc/sso-secret-volume
    secret/truststore-secret as truststore-secret
    mounted at /etc/truststore

    ```

7.  要使用命令行界面对单点登录进行配置更改，请创建一个名为`sso-extensions.cli`的文件，其内容如下:

    ```
    embed-server --std-out=echo --server-config=standalone-openshift.xml
    batch
    /system-property=javax.net.debug:add(value="ssl,handshake")
    /subsystem=infinispan/cache-container=keycloak:write-attribute(name=module,value=org.keycloak.keycloak-model-infinispan)
    /socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=remote-cache/:add(host=rhsso-infinispan.rhsso.svc.cluster.local,port=${remote.cache.port:11222},fixed-source-port=true)
    run-batch
    batch
    /subsystem=infinispan/cache-container=keycloak/replicated-cache=work/store=remote:add(cache=work,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,remoteStoreSecurityEnabled=true,statistics=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=sessions/store=remote:add(cache=sessions,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=offlineSessions/store=remote:add(cache=offlineSessions,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=clientSessions/store=remote:add(cache=clientSessions,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=offlineClientSessions/store=remote:add(cache=offlineClientSessions,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=loginFailures/store=remote:add(cache=loginFailures,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    /subsystem=infinispan/cache-container=keycloak/distributed-cache=actionTokens/store=remote:add(cache=actionTokens,remote-servers=[remote-cache],fetch-state=false,passivation=false,preload=false,purge=false,shared=true,properties={rawValues=true,marshaller=org.keycloak.cluster.infinispan.KeycloakHotRodMarshallerFactory,infinispan.client.hotrod.trust_store_file_name=/etc/truststore/truststore.jks,infinispan.client.hotrod.trust_store_password=redhat,infinispan.client.hotrod.sasl_mechanism=DIGEST-MD5,infinispan.client.hotrod.auth_username=developer,infinispan.client.hotrod.auth_password=JkSURXkBfLqsRG6M,infinispan.client.hotrod.use_ssl=true,remoteStoreSecurityEnabled=true,statistics=true,infinispan.client.hotrod.auth_realm=default,infinispan.client.hotrod.auth_server_name=infinispan})
    run-batch
    reload
    quit

    ```

8.  使用这个`sso-extensions.cli`文件创建一个配置图，并将其挂载为一个卷:

    ```
    $ oc create configmap jboss-cli --from-file=sso-extensions.cli
    $ oc set volume dc/sso --add --name=jboss-cli -m /opt/eap/extensions -t configmap --configmap-name=jboss-cli --default-mode='0755' --overwrite

    ```

9.  使用`JAVA_OPTS`:
    :

    ```
    $ oc set env dc/sso \
    -e "JAVA_OPTS_APPEND= \
    -Djboss.site.name=site1"

    ```

    在部署配置中定义站点名称
10.  要登录 SSO 控制台，请创建一个用户:

    ```
    sh-4.4$ ./add-user-keycloak.sh \
    > -r master \
    > -u admin \
    > -p password

    sh-4.4$ ./jboss-cli.sh --connect ':reload'

    ```

## 验证集成

最后一步是验证集成。首先，您将登录到 OpenShift 项目，然后您将在 SSO pod 中打开一个 shell。

### 登录到 OpenShift 项目

运行`oc get pods -o wide`获取 SSO pod 的 IP 地址。

### 打开 SSO pod 中的外壳

执行以下步骤来打开 SSO 窗格中的外壳。

1.  使用远程 shell 如下:

    ```
    $ oc rsh sso-49-scgz2

    ```

2.  转到`opt/eap/bin/`文件夹:

    ```
    $ cd opt/eap/bin/
    ```

3.  打开数据网格控制台，导航到`clientSessions`缓存。
4.  调用单点登录 CLI 并创建连接:

    ```
    $ ./kcadm.sh config credentials --server http://10.129.2.250:8080/auth --realm master --user admin --password password

    ```

5.  刷新数据网格控制台，观察到每次调用 SSO 命令行界面，`clientSessions`缓存中的条目数都会增加，如图 6 所示。
    [![](img/87021a53740def289b662c50a406dc23.png "verify")](/sites/default/files/blog/2021/03/verify.png)

请注意，如果在 SSO 端添加了任何用户，缓存条目也会增加。如果重新创建 SSO pod，数据不会丢失，缓存条目会保留在数据网格中。

## 结论

对于跨云平台的大型数据集，Red Hat 数据网格是一个有价值的解决方案。数据网格为 OpenShift 上的混合云部署提供了许多特性。在本文中，您看到了如何将单点登录技术客户端与作为远程存储的数据网格服务器连接起来，然后将它们部署在 Red Hat OpenShift 上。

*Last updated: April 22, 2021*
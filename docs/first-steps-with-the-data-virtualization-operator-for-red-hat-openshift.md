# Red Hat OpenShift 数据虚拟化操作员的第一步

> 原文：<https://developers.redhat.com/blog/2020/01/21/first-steps-with-the-data-virtualization-operator-for-red-hat-openshift>

Red Hat Integration [Q4 版本](https://www.redhat.com/en/blog/whats-new-red-hat-integration)增加了许多新特性和功能，并日益关注云原生数据集成。我最感兴趣的特性是[模式注册中心](https://developers.redhat.com/blog/2019/11/26/red-hat-simplifies-transition-to-open-source-kafka-with-new-service-registry-and-http-bridge/)的引入，基于 [Debezium 到技术预览版](https://developers.redhat.com/blog/2019/11/22/red-hat-advances-debezium-cdc-connectors-for-apache-kafka-support-to-technical-preview/)的变更数据捕获能力的提升，以及数据虚拟化(技术预览版)能力。

数据集成是一个到目前为止还没有得到云原生社区太多关注的话题，我们将在未来的帖子中更详细地讨论它。在这里，我们直接进入在 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 4 上演示最新发布的[数据虚拟化](https://access.redhat.com/documentation/en-us/red_hat_integration/2019-12/html-single/data_virtualization/index) (DV)功能。这是一个循序渐进的可视化教程，描述了如何使用 Red Hat Integration 的数据虚拟化操作符创建一个简单的虚拟数据库。本教程结束时，您将学会:

*   如何部署 DV 操作器？
*   如何创建虚拟数据库？
*   如何访问虚拟数据库？

本文中的步骤适用于任何有操作员支持的 Openshift 4.x 环境，甚至是时间和资源受限的环境，如 [Red Hat OpenShift 交互式学习门户](https://learn.openshift.com/)。

## 准备环境

首先，你需要准备一个在 OpenShift 4 上部署产品化 DV 操作器的环境。如果您是 Red Hat 合作伙伴，并且可以访问 Red Hat 产品演示系统(RHPDS)，请从 Services 菜单中请求一个“OpenShift 4 Workshop”类型的 OpenShift 集群，如图 1 所示。一个包含一个节点(五个用户)的简单集群应该足够了:

[![](img/51048de6d18b91a09b10ce3c6a522e60.png "img_5df0bf78aa256")](/sites/default/files/blog/2019/12/img_5df0bf78aa256.png)

**备选方案**:无论您是否是合作伙伴，如果您无法访问 RHPDS，同样的步骤也适用于[红帽 OpenShift 互动学习门户](https://learn.openshift.com/)。在那里，您将获得一个资源和时间受限的 OpenShift 集群一个小时。在这种环境下，选择 *OpenShift 4.2 Playground* 并遵循相同的步骤(并且不要忘记快速键入)。本指南的其余部分假设 RHPDS 为环境。

现在，创建`dv-demo`项目。

### 在 GUI 中

设置好 OpenShift 集群后，登录 OpenShift 并转到项目页面，如图 2 所示:

[![The OpenShift Projects page.](img/5d777546c0f2fb148860200ca75ad61d.png "img_5df0bfbecb1b1")](/sites/default/files/blog/2019/12/img_5df0bfbecb1b1.png)

Figure 2: Create your new project from the OpenShift Projects page.

创建一个名为`dv-demo`的新项目。

### 在命令行上

或者，获取您的令牌并从命令行登录，创建一个名为`dv-demo`的新项目:

```
$ oc login --token=YOUR_TOKEN --server=YOUR_SERVER
$ oc new-project dv-demo
```

然后，在`dv-demo`名称空间中执行以下所有操作。使用您的 Red Hat 客户入口网站凭证创建一个密码，以访问 Red Hat 容器图像:

```
$ oc create secret docker-registry dv-pull-secret --docker-server=registry.redhat.io --docker-username=$RH_PORTAL_USERNAME --docker-password=$RH_PORTAL_PASSWORD --docker-email=$RH_PORTAL_EMAIL
$ oc secrets link builder dv-pull-secret
$ oc secrets link builder dv-pull-secret --for=pull
```

## 部署 DV 操作员

以下步骤部署 DV 操作员:

1.  从 OpenShift 菜单中选择*目录- >操作员中枢*。
2.  搜索“数据虚拟化”
3.  Select the *Data Virtualization Operator* 7.5.0 provided by Red Hat, Inc., as shown in Figure 3:
    [![](img/f97055d49303352ac90a18e39ee629a4.png "img_5df0c01e17f91")](/sites/default/files/blog/2019/12/img_5df0c01e17f91.png)

    Figure 3: The Data Virtualization Operator dialog box.

    **备选:**由 Red Hat，Inc .提供的数据虚拟化 Operator 7.5.0 需要凭证才能访问 Red Hat 容器目录。相反，您可以使用基于上游容器图像的 Teiid Community 操作符。它的实现不需要 Red Hat 凭证。Red Hat 不支持该操作符，但它可以用于快速演示。

4.  点击*安装。*
5.  确保名称空间与之前创建的名称空间相同，然后点击*订阅*，如图 4:
    [![The Create Operator Subscription dialog box.](img/13a29b319f7318bc071beccdcb658ea4.png "img_5df0c043cf990")](/sites/default/files/blog/2019/12/img_5df0c043cf990.png)

    图 4:创建运营商订阅对话框。

    

几分钟后，您应该看到 DV Operator 作为 pod 部署在运行状态，如图 5 所示:

[![](img/c38cc10022036523b11c0bd030b06e41.png "img_5df0c0c8a8241")](/sites/default/files/blog/2019/12/img_5df0c0c8a8241.png)

Figure 5: The DV Operator as shown in the Pods listing.

6.  将之前创建的秘密与`dv-operator`服务帐户关联起来，这样 DV 操作员也可以从 Red Hat 注册表中提取图像。该步骤应该在安装了操作员并创建了服务帐户之后，在创建新的虚拟数据库 CustomResource (CR):

    ```
    $ oc secrets link dv-operator dv-pull-secret --for=pull
    ```

    之前*执行*

至此，操作员安装过程完成，可以创建虚拟化了。

## 创建一个示例数据库

在创建虚拟化之前，我们需要一个填充了数据的示例数据库。以下脚本安装 PostgreSQL 数据库并插入示例数据。如果您要为现有数据库创建虚拟化，则可以跳过这一步，转到下一节:

1.  创建 PostgreSQL 数据库:

    ```
    $ oc new-app 
    -e POSTGRESQL_USER=user 
    -e POSTGRESQL_PASSWORD=mypassword 
    -e POSTGRESQL_DATABASE=sampledb 
    postgresql:9.6
    ```

2.  一旦 PostgreSQL pod 处于运行状态，连接并运行`psql`客户端:

    ```
    $ oc rsh $(oc get pods -o name -l app=postgresql)
    $ psql -U user sampledb
    ```

3.  创建一些表格并用数据填充它们:

    ```
    CREATE TABLE CUSTOMER (
    ID bigint,
    SSN char(25),
    NAME varchar(64),
    CONSTRAINT CUSTOMER_PK PRIMARY KEY(ID));

    CREATE TABLE ADDRESS (
    ID bigint,
    STREET char(25),
    ZIP char(10),
    CUSTOMER_ID bigint,
    CONSTRAINT ADDRESS_PK PRIMARY KEY(ID),
    CONSTRAINT CUSTOMER_FK FOREIGN KEY (CUSTOMER_ID) REFERENCES CUSTOMER (ID));
    INSERT INTO CUSTOMER (ID,SSN,NAME) VALUES (10, 'CST01002','Joseph Smith');
    INSERT INTO CUSTOMER (ID,SSN,NAME) VALUES (11, 'CST01003','Nicholas Ferguson');
    INSERT INTO CUSTOMER (ID,SSN,NAME) VALUES (12, 'CST01004','Jane Aire');
    INSERT INTO ADDRESS (ID, STREET, ZIP, CUSTOMER_ID) VALUES (10, 'Main St', '12345', 10);
    q
    exit
    ```

至此，我们已经建立了一个简单的数据库，并为我们的演示做好了准备。

## 创建虚拟数据库

现在，我们已经安装了操作符，并填充了一个示例数据库。我们已经准备好创建虚拟化。这里有几种方法。

### 最简单的方法

为此，我们将使用最简单的方法，将数据描述语言(DDL)定义嵌入到 CR 本身中。对于从外部位置检索 DDL 及其依赖项的其他方法，请查看本文档末尾的“后续步骤”部分:

1.  从 OpenShift 菜单中选择**目录**->-**已安装操作符**如图 6:

*[![OpenShift's Catalog -&gt; Install Operators dialog box.](img/5cb108a366e73bf47906595e6f73b41d.png "img_5df0c104b173c")](/sites/default/files/blog/2019/12/img_5df0c104b173c.png)* *2.  选择**数据虚拟化操作员**。
3.  选择**虚拟数据库**选项卡。
4.  点击**创建虚拟数据库**，打开**创建虚拟数据库**对话框，如图 7 所示:

[![The Create Virtual Database dialog box.](img/4f065f5a514d7bc1a8839a0a4fb97167.png "img_5df0c17be532e")](/sites/default/files/blog/2019/12/img_5df0c17be532e.png)

5.  检查并修改定义数据虚拟化的示例 CR。

虚拟化的本质是 DDL 定义，它使用 SQL-MED 规范在物理数据层之上创建新的抽象数据层。下面是该示例的简要说明:

```
//Step 1 Defines a name for the virtual database you want to create
CREATE DATABASE customer OPTIONS (ANNOTATION 'Customer VDB');USE DATABASE customer;

//Step 2: Configures a translator to interpret data from the datasource
CREATE FOREIGN DATA WRAPPER postgresql;

//Step 3: Configures the datasource connection details for the external source
CREATE SERVER sampledb TYPE 'NONE' FOREIGN DATA WRAPPER postgresql;

//Step 4: Creates schemas to hold metadata about the source and virtual layers
CREATE SCHEMA accounts SERVER sampledb;
CREATE VIRTUAL SCHEMA portfolio;

//Step 5: Imports the metadata from source schema into a virtual schema
SET SCHEMA accounts;
IMPORT FOREIGN SCHEMA public FROM SERVER sampledb INTO accounts OPTIONS("importer.useFullSchemaName" 'false');

// Step 6: Create virtual views
SET SCHEMA portfolio;
CREATE VIEW CustomerZip(id bigint PRIMARY KEY, name string, ssn string, zip string) AS
   SELECT c.ID as id, c.NAME as name, c.SSN as ssn, a.ZIP as zip
FROM accounts.CUSTOMER c LEFT OUTER JOIN accounts.ADDRESS a
ON c.ID = a.CUSTOMER_ID;
```

要了解关于这些结构的更多信息，请查看数据虚拟化指南。

**备选项 *:*** 在创建虚拟数据库之前，您可以定制包含虚拟化和嵌入式 DDL 细节的 CR。在上面的 CR 示例中，连接细节是硬编码的，并且与前面创建的 PostgreSQL 数据库的细节相匹配。如果你只是想运行这个例子，不要编辑任何东西，点击**创建**按钮。如果您更喜欢从 Kubernetes Secret 对象中检索 PostgreSQL 连接细节，而不是将它们硬编码到 CR 中，请参阅“使用 Kubernetes Secret 对象”一节

### 使用 Kubernetes 的秘密对象

要使用 Kubernetes 秘密对象:

1.  创建将用于从虚拟化访问数据库的密码:

```
oc create -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: postgresql
type: Opaque
stringData:
  database-user: user
  database-name: sampledb
  database-password: mypassword
EOF
```

2.  在 CR 中，替换:

```
- name: SPRING_DATASOURCE_SAMPLEDB_USERNAME
value: user
- name: SPRING_DATASOURCE_SAMPLEDB_PASSWORD
value: mypassword
- name: SPRING_DATASOURCE_SAMPLEDB_DATABASENAME
value: sampledb
```

使用:

```
- name: SPRING_DATASOURCE_SAMPLEDB_USERNAME
valueFrom:
secretKeyRef:
name: postgresql
key: database-user
- name: SPRING_DATASOURCE_SAMPLEDB_PASSWORD
valueFrom:
secretKeyRef:
name: postgresql
key: database-password
- name: SPRING_DATASOURCE_SAMPLEDB_DATABASENAME
valueFrom:
secretKeyRef:
name: postgresql
key: database-name
```

3.  点击 **Create** 按钮实例化虚拟化，如图 8 所示:

[![Creating the alternative virtual database.](img/f3228faf053760df4f2fef5cc5cbf456.png "img_5df0c27038ac2")](/sites/default/files/blog/2019/12/img_5df0c27038ac2.png)

Figure 8: Creating the alternative virtual database.

几分钟后，您应该会看到除了`dv-operator`和`postgresql`pod 之外，还有一个`rdbms-springboot` pod 正在运行我们的新虚拟化。一旦这个 pod 处于运行状态，我们就可以使用新的虚拟化了，如图 9 所示:

[![](img/16a2d9701ce5fead7f7117d354698c9f.png "img_5df0c2d598af2")](/sites/default/files/blog/2019/12/img_5df0c2d598af2.png)

### 使用 CLI

虚拟化不仅可以从前面部分演示的 OpenShift 控制台创建，还可以作为 CI/CD 过程的一部分通过命令行创建。您可以使用一个 CLI 命令创建一个类似的虚拟化(名为`dv-customer`)，如下所示:

```
$ oc create -f https://raw.githubusercontent.com/teiid/teiid-openshift-examples/7.5-1.2.x/rdbms-example/dv-customer.yaml
```

最后，你可以混合搭配你的方法。通过 CLI 创建您的虚拟化，然后从 OpenShift 控制台或反过来管理它们。完成后，您还可以在 CLI 中删除虚拟化，如下所示:

```
$ oc delete vdb dv-customer
```

## 访问虚拟数据库

除了部署虚拟化之外，DV Operator 还创建了一个服务和一个路由来公开各种虚拟数据库端点。

### 从集群内部访问

要查看该服务公开了哪些协议，请进入 OpenShift 菜单，选择**联网** - > **服务**。然后，选择名为`rdbms-springboot`的服务，如图 10 所示:

[![The rdbms-springboot service overview window.](img/5fb3b091228419d72919c912b37e3f54.png "img_5df0c30fdd7d5")](/sites/default/files/blog/2019/12/img_5df0c30fdd7d5.png)

这里命名的所有端口都可以从 OpenShift 集群内部访问；例如，从想要使用虚拟化的其他 pod。在这里，我们可以看到列出了以下端点:

### 从群集外部进行 HTTP 访问

从 OpenShift 集群外部访问虚拟化最简单的方法是通过 HTTP 和 OData/OpenAPI。要找到创建的路由，进入 OpenShift 菜单，选择*联网- >路由，*，选择以虚拟化`rdbms-springboot`命名的路由，如图 11 所示:

[![The rdbms-springboot route in the OpenShift interface.](img/cf08af77a3d856be5262fe72ddfe2f98.png "img_5df0c39f320c0")](/sites/default/files/blog/2019/12/img_5df0c39f320c0.png)

单击该位置将显示“404 -未找到”，因为“/”处没有端点。要查看 OData 元数据，请将`odata/$metadata`附加到 URL 的末尾(例如，`https://rdbms-springboot-dv-demo.example.com/odata/$metadata`)。

如果不想编辑 URL，想直接访问 OData 端点，进入 OpenShift 菜单，选择*Installed Operator->DV Operator->虚拟数据库- > rdbms-springboot* ，然后路由链接。

基于 [OData 基础教程](https://www.odata.org/getting-started/basic-tutorial/)尝试其他查询，例如:

*   检索按姓名排序的前 2 条客户记录:
    `https://rdbms-springboot-dv-demo.example.com/odata/portfolio/CustomerZip?$top=2&$orderby=name`
*   通过 ID 检索客户记录，并选择名称字段值:
    `https://rdbms-springboot-dv-demo.example.com/odata/portfolio/CustomerZip(10)/name/$value`
*   按姓名搜索客户记录，并在 JSON 中显示结果:
    `https://rdbms-springboot-dv-demo.example.com/odata/portfolio/CustomerZip?$filter=startswith(name,%27Joseph%27)&$format=JSON`
*   从 OData 元数据
    `https://rdbms-springboot-dv-demo.example.com/odata/openapi.json?version=3`生成 OpenAPI v3 模式

### 从集群外部进行 JDBC/ODBC 访问

为了从 OpenShift 集群外部访问 JDBC/ODBC 上的虚拟化，我们必须向外界公开 JDBC/ODBC 端口。然后，我们可以使用支持这些协议的 SQL 客户端，或者使用这个基于 Spring Boot 的[客户端应用程序](https://github.com/bibryam/teiid-examples/tree/master/client)。

#### 最简单的方法

出于开发目的(特别是)公开 JDBC/ODBC 端口的最简单方法是使用如下的`oc`客户端:

```
$ oc port-forward $(oc get pods -o=jsonpath='{.items[0].metadata.name}' -l app=rdbms-springboot) 35432 31000
```

该命令将标签为`rdbms-springboot`的 pod 的端口 35432 (ODBC/PG)和 31000(teid JDBC)映射到本地机器。当此命令在终端中运行时，您可以启动一个应用程序，并使用如下配置连接到该应用程序:

```
jdbc:postgresql://127.0.0.1:35432/customer?sslMode=disable
```

或者

```
jdbc:teiid:customer@mm://127.0.0.1:31000
```

#### 创建 Kubernetes 负载平衡器服务

公开同一个端口的另一种方法是创建一个 Kubernetes LoadBalancer 服务，可以用粘性会话等进行定制。要创建负载平衡器服务，请执行以下操作:

```
$ oc apply -f - << EOF
apiVersion: v1
kind: Service
metadata:
  name: rdbms-springboot-expose
spec:
  type: LoadBalancer
  ports:
  - name: teiid
    port: 31000  
  - name: pg
    port: 35432          
  selector:
    app: rdbms-springboot
EOF
```

创建服务后，您可以通过以下查询发现完整的 URL，以访问集群和基础设施提供者的负载平衡:

```
$ oc get svc rdbms-springboot-expose -o=jsonpath='{..ingress[0].hostname}'
```

对于我的 OpenShift 集群，该命令返回以下 URL:

```
A4d3bf5fd1b9311eab2b602474b8b0b4-143190945.eu-central-1.elb.amazonaws.com
```

上面的示例程序或任何其他工具都可以使用该 URL，如下所示:

```
jdbc:postgresql://a4d3bf5fd1b9311eab2b602474b8b0b4-143190945.eu-central-1.elb.amazonaws.com:35432/customer?sslMode=disable
```

或者

```
jdbc:teiid:customer@mm://a4d3bf5fd1b9311eab2b602474b8b0b4-143190945.eu-central-1.elb.amazonaws.com:31000
```

上述机制使得从 OpenShift 集群外部访问正在运行的虚拟化变得容易。还有其他机制，但这些对于入门体验来说已经足够了。

## 后续步骤

恭喜您，您已经到达如何开始使用 OpenShift 的数据虚拟化操作符的末尾。数据虚拟化是 Red Hat 集成堆栈的重要组成部分，我们将在以后的文章中介绍它如何与其他 Red Hat 技术集成。要了解有关数据虚拟化的更多信息，请尝试我们的[示例](https://github.com/teiid/teiid-openshift-examples)并查看最新的 Red Hat Integration [文档](https://access.redhat.com/documentation/en-us/red_hat_integration/2019-12/)，了解如何使用其他数据源配置数据虚拟化并通过 3scale 保护其端点。

如果您有与运行数据虚拟化技术预览相关的请求或问题，请发送电子邮件至[数据集成预览邮件列表](mailto:data-integration-preview@redhat.com)告知我们。

*Last updated: June 29, 2020**
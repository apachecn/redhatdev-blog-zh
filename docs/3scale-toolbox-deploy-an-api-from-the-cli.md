# 3 缩放工具箱:从 CLI 部署 API

> 原文：<https://developers.redhat.com/blog/2019/07/29/3scale-toolbox-deploy-an-api-from-the-cli>

[从 CI/CD 管道中部署 API](https://developers.redhat.com/blog/?p=608917)可能会有大量的工作。最新发布的 Red Hat Integration 通过向 [3scale](https://developers.redhat.com/products/3scale/overview) CLI 添加新功能，极大地改善了这种情况。3scale CLI 被命名为 [3scale toolbox](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.4/html/api_devops/cli-toolbox) ，致力于帮助 API 管理员运营他们的服务，并通过连续交付管道自动交付他们的 API。

拥有标准 CLI 对我们的客户来说是一大优势，因为他们可以在自己选择的 CI/CD 解决方案(Jenkins、GitLab CI、Ansible、Tekton 等)中使用它。).这也是 Red Hat 尽可能多地捕捉客户需求并向所有客户提供相同功能集的一种方式。

## 3 缩放工具箱概览

3scale 工具箱可以管理多种对象:

*   **服务**:从 CSV 或 OpenAPI 规范文件导入。
*   **代理:**将代理配置升级到生产，显示当前配置。
*   **ActiveDocs** :管理 OpenAPI 规范库。
*   **策略**:将策略从一个注册表复制到另一个注册表。
*   **方法**和**指标。**
*   **应用计划**:从 artefact 文件导入应用计划。
*   **应用**:管理客户端应用。
*   **账户**:管理客户账户。

3scale 工具箱遵循 CLI 的常规惯例:

*   出错时输出非零状态代码。
*   stderr 包含错误消息；stdout 包含有用的输出。
*   对于需要由脚本或管道解析的数据，3scale 工具箱可以输出 JSON 或 YAML。

对于这些约定，我们添加了一个关键原则:大多数操作应该是[幂等的](https://en.wikipedia.org/wiki/Idempotence)。这意味着您可以简单地陈述您希望系统如何运行，3scale 工具箱将相应地采取行动:如果存在，更新现有配置；如果缺少，则创建。幂等性将帮助您在出现中断或瞬时扰动时构建更可靠管道。

## 支持状态

3scale 工具箱是 Red Hat 集成解决方案的受支持组件。它在 Red Hat Enterprise Linux (RHEL)和 OpenShift 上受到本机支持。在 RHEL 上，工具箱由通道“rhel-7-server-3 scale-amp-2.6-rpms”附带的 RPM“3 scale-toolbox”提供在 OpenShift 上，可以使用容器图像“3scale-amp26/toolbox”。

在此版本中，我们将 Jenkins 作为 3scale 工具箱的主要用例，但只要工具箱[在支持的配置](https://access.redhat.com/articles/2798521)上运行，您就可以将它用于任何其他 CI/CD 解决方案。

## 目标使用案例

3scale 工具箱可用于实现各种各样的使用案例:

*   CI/CD 管道:从您首选的 CI/CD 解决方案中持续部署您的 API。
*   灾难恢复计划:3scale 工具箱可以将现有服务和策略从一个实例复制到另一个实例。
*   一次性脚本:例如，在迁移之前自动删除未使用的服务。

## 3 秤工具箱的安装

通过运行以下命令，在您的 Red Hat Enterprise Linux 服务器上安装工具箱:

```
$ sudo yum install --enablerepo=rhel-7-server-3scale-amp-2.6-rpms 3scale-toolbox
```

您可以通过执行以下命令来确认工具箱已安装:

```
$ 3scale --version
0.12.3
```

## 3 秤工具箱的配置

要使用工具箱，您必须生成一个对帐户管理 API 具有写权限的访问令牌。然后，您可以添加一个“遥控器”:

```
$ 3scale remote add 3scale-saas "https://123...456@MY-TENANT-admin.3scale.net/"
```

你必须更换“123...456”和您的 3scale 管理门户的名称“MY-TENANT”。

您可以通过列出现有服务(必须至少有一个)来确认配置正在工作:

```
$ 3scale service list 3scale-saas
ID             NAME      SYSTEM_NAME
2555417757658  Echo API  api
```

## 简单的用例:从 CLI 部署 API

对于第一次联系，让我们选择一个非常简单的用例:我们想从 CLI 部署一个 API，并确保它端到端地工作。

首先，获取啤酒目录服务的 OpenAPI 规范文件:

```
$ curl -sfk -o swagger.json https://raw.githubusercontent.com/microcks/api-lifecycle/master/beer-catalog-demo/api-contracts/beer-catalog-api-swagger.json
```

部署新服务:

```
$ 3scale import openapi -d 3scale-saas swagger.json --override-private-base-url=https://echo-api.3scale.net -t beer-catalog
Created service id: 2555417822198, name: Beer Catalog API
Service proxy updated
destroying all mapping rules
Created GET /beer/{name}$ endpoint
Created GET /beer/findByStatus/{status}$ endpoint
Created GET /beer$ endpoint
```

创建应用程序计划:

```
$ 3scale application-plan apply 3scale-saas beer-catalog test -n "Test Plan" --default Applied application plan id: 2357356113164; Default: true
```

创建应用程序:

```
$ 3scale application apply 3scale-saas 1234567890abcdef --account=john --name="Test Application" --description="Created from the CLI" --plan=test --service=beer-catalog
Applied application id: 1409618501689
```

获取临时网关的 URL:

```
$ STAGING_URL=$(3scale proxy-config show 3scale-saas beer-catalog sandbox |jq -r .content.proxy.sandbox_endpoint)
$ echo $STAGING_URL
https://beer-catalog-2445582535750.staging.gw.apicast.io:443 
```

**注意:**为了让这个命令成功，你需要安装[jq](https://stedolan.github.io/jq/download/)。

确保新部署的 API 正在工作:

```
$ curl -D - -H "api-key: 1234567890abcdef" "$STAGING_URL/beer"
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 730
Connection: keep-alive

{
  "method": "GET",
  "path": "/beer",
  ...
}
```

由于临时网关运行良好，现在您可以将配置升级到生产网关:

```
$ 3scale proxy-config promote 3scale-saas beer-catalog
Proxy Configuration version 2 promoted to 'production'
```

恭喜，您的 API 现在已经启动并运行了！

## 在容器中运行工具箱

在前面的例子中，我们在 Red Hat Enterprise Linux 服务器上运行工具箱，但是您也可以在容器中运行它！

第一步是创建一个包含工具箱配置的 Kubernetes 秘密:

```
$ oc create secret generic 3scale-toolbox --from-file="$HOME/.3scalerc.yaml"
secret/3scale-toolbox created
```

创建包含要部署的 OpenAPI 规范文件的配置映射:

```
$ oc create configmap openapi --from-file=swagger.json
configmap/openapi created
```

通过创建 Kubernetes 作业来运行工具箱:

```
$ oc create -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: toolbox
spec:
  backoffLimit: 0
  activeDeadlineSeconds: 300
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: toolbox
        image: quay.io/redhat/3scale-toolbox:master
        imagePullPolicy: Always
        args: [ "3scale", "import", "openapi", "-d", "3scale-saas", "/artifacts/swagger.json", "--override-private-base-url=https://echo-api.3scale.net", "-t", "beer-catalog" ]
        env:
        - name: HOME
          value: /config
        volumeMounts:
        - name: toolbox-config
          mountPath: /config
        - name: artifacts
          mountPath: /artifacts
      volumes: 
      - name: toolbox-config
        secret: 
          secretName: 3scale-toolbox
      - name: artifacts
        configMap: 
          name: openapi
EOF
```

**注意:**上面的工作定义使用了社区图像。红帽客户可以用这个图代替:3scale-amp26/toolbox。

确认啤酒目录服务已经更新:

```
$ oc logs -f job/toolbox 
Updated service id: 2555417822198, name: Beer Catalog API
Service proxy updated
destroying all mapping rules
Created GET /beer/{name}$ endpoint
Created GET /beer/findByStatus/{status}$ endpoint
Created GET /beer$ endpoint
Activedocs exists, update!
```

## 结论

3scale 工具箱是许多涉及 API 和 Red Hat 集成解决方案的自动化的基础。它是一个受支持的组件，涵盖了广泛的使用案例。

了解 Red Hat Integration 的 API 管理功能如何帮助您从 CI/CD 管道部署 API:

*   [从 CI/CD 管道部署 API 的 5 个原则](https://developers.redhat.com/blog/?p=608917)
*   [从 Jenkins 管道部署您的 API](https://developers.redhat.com/blog/?p=612387)
*   [使用 3scale 工具箱詹金斯共享库](https://developers.redhat.com/blog/?p=612407)

*Last updated: January 21, 2022*
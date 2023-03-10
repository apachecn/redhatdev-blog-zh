# 从 Jenkins 管道部署您的 API

> 原文：<https://developers.redhat.com/blog/2019/07/30/deploy-your-api-from-a-jenkins-pipeline>

在上一篇文章[从 CI/CD 管道部署 API 的 5 个原则](https://developers.redhat.com/blog/?p=608917)中，我们发现了从 CI/CD 管道部署 API 所需的主要步骤，这可能是一个巨大的工作量。Red Hat Integration 的最新版本有望通过向 3scale CLI 添加新功能来大大改善这种情况。在[3scale toolbox:Deploy a API from CLI](https://developers.redhat.com/blog/?p=611307)中，我们发现了 3 scale toolbox 如何努力实现 API 交付的自动化。在本文中，我们将讨论 3scale 工具箱如何帮助您在[Red Hat open shift](https://developers.redhat.com/openshift/)/Kubernetes 上部署来自 Jenkins 管道的 API。

## 从 Jenkins 运行 3scale 工具箱

从 Jenkins 管道中调用 CLI 时，至少有三种不同的策略:

*   您可以创建包含 CLI 的自定义 Jenkins 从属映像。
*   您可以在管道运行期间动态安装 CLI。
*   或者，您可以启动 Kubernetes 作业或 Pod，在容器中调用 CLI。

自定义 Jenkins 从属服务器的缺点是，它是另一个需要构建、管理、故障排除、保护等的容器映像。这也意味着凭证需要从 Jenkins 内部管理，这需要一个专有插件来应用 RBAC。动态安装 CLI 适用于静态二进制文件，如 jq 或 oc 命令，但不适用于 Ruby 程序。此外，证书需要在 Jenkins 内部进行管理。

Kubernetes Job/Pod 解决方案的优势在于消除了任何依赖性/准备工作。它允许在 OpenShift/Kubernetes 中进行凭证管理(因此受制于 RBAC ),并且是面向未来的，因为这是 Tekton 管道的工作方式。

要从 Jenkins 管道部署 API，请以 Kubernetes 作业的身份运行 3scale 工具箱。

## 从 Jenkins 运行 3scale 工具箱

要从 Jenkins 管道以 Kubernetes 作业的形式运行 3scale 工具箱，我们需要编写一个小的助手函数:

```
def runToolbox(args) {
  def kubernetesJob = [
    "apiVersion": "batch/v1",
    "kind": "Job",
    "metadata": [
      "name": "toolbox"
    ],
    "spec": [
      "backoffLimit": 0,
      "activeDeadlineSeconds": 300,
      "template": [
        "spec": [
          "restartPolicy": "Never",
          "containers": [
            [
              "name": "job",
              "image": "quay.io/redhat/3scale-toolbox:master",
              "imagePullPolicy": "Always",
              "args": [ "3scale", "version" ],
              "env": [
                [ "name": "HOME", "value": "/config" ]
              ],
              "volumeMounts": [
                [ "mountPath": "/config", "name": "toolbox-config" ],
                [ "mountPath": "/artifacts", "name": "artifacts" ]
              ]
            ]
          ],
          "volumes": [
            [ "name": "toolbox-config", "secret": [ "secretName": "3scale-toolbox" ] ],
            [ "name": "artifacts", "configMap": [ "name": "openapi" ] ]
          ]
        ]
      ]
    ]
  ]

  kubernetesJob.spec.template.spec.containers[0].args = args

  sh "rm -f -- job.yaml"
  writeYaml file: "job.yaml", data: kubernetesJob
  sh """
  oc delete job toolbox --ignore-not-found
  sleep 2
  oc create -f job.yaml
  sleep 20 # Adjust the sleep duration to your server velocity
  """

  def logs = sh(script: "oc logs -f job/toolbox", returnStdout: true)
  echo logs
  return logs
}
```

该功能首先定义一个 Kubernetes 作业模板，您可以根据自己的环境对其进行定制。特别是，您可以调整:

*   *backoffLimit* :在管道开发期间保持为 0，一旦你的管道准备好使用，就提高到 2。补偿限制决定了出错时将执行多少次尝试。
*   *activeDeadlineSeconds*:300 秒应该足够完成一个工具箱的执行，但是如果你的服务器很慢、负载很重或者你的 API 包含很多操作，你可能需要将它设置为一个更高的值。
*   *图片*:你可以使用 [quay.io](https://quay.io/repository/redhat/3scale-toolbox?tag=latest&tab=tags) 上的社区图片，如果你是红帽的客户，也可以使用红帽官方图片(3scale-amp26/toolbox)。

然后用提供的命令行参数修补作业模板。生成的作业存储为 YAML 文件，并使用 oc 命令部署在 OpenShift 中。就在 oc 命令之后，管道等待几秒钟，让 Kubernetes 作业创建容器，并让它从“ContainerCreated”状态转换到“Running”状态。您可以根据您的服务器速度来调整这个等待时间，或者更好的是，用轮询循环来代替它。最后，一旦容器转换到“运行中”状态，该函数获取其日志并将其返回给调用者。

如您所见，该作业接收两个装载点:

*   包含远程列表的 3scale 工具箱配置文件的 Kubernetes 机密。
*   要供应的工件的 Kubernetes 配置图(OpenAPI 规范文件、应用程序计划文件等)。).

配置映射将由 Jenkins 管道创建。秘密是在管道之外提供的，并且受 RBAC 的约束。

**在与 Jenkins Master 相同的 OpenShift 项目中提供 3scale-toolbox secret** :

```
3scale remote add 3scale-instance "https://123...456@MY-TENANT-admin.3scale.net/"
oc create secret generic 3scale-toolbox --from-file="$HOME/.3scalerc.yaml"

```

**注意:**在本文的其余部分，我们将展示一个带有 hosted APIcast 的 3 规模托管实例。当使用自我管理的 APIcast 或本地实例时，可能需要进行一些小的调整。

## 准备一个非常简单的 Jenkins 管道来部署您的 API

在您的 Jenkins 主服务器上，创建一个新的管道，并使用一些全局变量(您稍后可以将其提升为管道参数)启动管道脚本:

```
def targetSystemName = "saas-usecase-apikey"
def targetInstance = "3scale-instance"
def privateBaseURL = "http://echo-api.3scale.net"
def testUserKey = "azerty1234567890"
def developerAccountId = "john"
def publicStagingBaseURL = null
def publicProductionBaseURL = null
```

这些变量直接映射到众所周知的 3 标度概念:

*   *目标系统名*是要创建的服务的标识符(系统名)。
*   *targetInstance* 是您在上面创建的工具箱遥控器的名称。
*   *privateBaseURL* 是服务的私有基础 URL。
*   *testUserKey* 是将用于执行端到端测试的用户密钥/API 密钥。
*   *developerAccountId* 是默认“开发人员”帐户的标识符(或其管理员的用户名:john)。
*   *publistagingbaseurl*是服务的公共临时基础 URL。
*   *publicProductionBaseURL* 是服务的公共生产基础 URL。

**注意:**如果您使用自我管理的 APIcast 或 3scale 的本地安装，您需要为 publicStagingBaseURL 和 publicProductionBaseURL 变量提供一个适当的值。

## 詹金斯管道阶段部署您的 API

在这一节中，我们将实现上一篇文章中描述的管道阶段:[从 CI/CD 管道](https://developers.redhat.com/blog/?p=608917)部署 API 的 5 个原则。

### 获取 OpenAPI 规范文件

添加一个管道阶段来获取您的 OpenAPI 规范文件，并在 OpenShift 上将其作为 ConfigMap 提供:

```
node() {
  stage("Fetch OpenAPI") {
    sh """
    curl -sfk -o swagger.json https://raw.githubusercontent.com/microcks/api-lifecycle/master/beer-catalog-demo/api-contracts/beer-catalog-api-swagger.json
    oc delete configmap openapi --ignore-not-found
    oc create configmap openapi --from-file="swagger.json"
    """
  }
```

OpenAPI 规范包含以 3 种规模提供 API 合同所需的规范:

*   服务元数据，如名称、版本、维护者等。
*   有效负载模式。
*   操作列表。
*   用于保护服务的安全方案。

### 导入 OpenAPI 规范文件

添加一个使用 3scale 工具箱将 OpenAPI 规范文件导入 3scale 的管道阶段:

```
  stage("Import OpenAPI") {
    def tooboxArgs = [ "3scale", "import", "openapi", "-d", targetInstance, "/artifacts/swagger.json", "--override-private-base-url=${privateBaseURL}", "-t", targetSystemName ]
    if (publicStagingBaseURL !=null) {
      tooboxArgs += "--staging-public-base-url=${publicStagingBaseURL}"
    }
    if (publicProductionBaseURL !=null) {
      tooboxArgs += "--production-public-base-url=${publicProductionBaseURL}"
    }
    runToolbox(tooboxArgs)
  }
```

这个阶段使用一些 groovy sugar 来动态地设置工具箱命令行参数，如果设置了公共的 Staging 或 Production Base URLs 的话。

### 创建应用程序计划和应用程序

添加使用工具箱创建 3 规模应用程序计划和应用程序的管道阶段:

```
  stage("Create an Application Plan") {
    runToolbox([ "3scale", "application-plan", "apply", targetInstance, targetSystemName, "test", "-n", "Test Plan", "--default" ])
  }

  stage("Create an Application") {
    runToolbox([ "3scale", "application", "apply", targetInstance, testUserKey, "--account=${developerAccountId}", "--name=Test Application", "--description=Created by Jenkins", "--plan=test", "--service=${targetSystemName}" ])
  }
```

### 运行端到端测试

添加一个使用工具箱运行端到端测试的阶段。要在使用 3 个规模托管实例时运行端到端测试，您必须获取代理定义以提取临时公共 URL。否则，您可以直接重用上面定义的 publicStagingBaseURL 变量:

```
  stage("Run integration tests") {
    if (publicStagingBaseURL == null) {
      def proxyDefinition = runToolbox([ "3scale", "proxy", "show", targetInstance, targetSystemName, "sandbox" ])
      def proxy = readJSON text: proxyDefinition
      publicStagingBaseURL = proxy.content.proxy.sandbox_endpoint
    }

    sh """
    echo "Public Staging Base URL is ${publicStagingBaseURL}"
    echo "userkey is ${testUserKey}"
    curl -vfk ${publicStagingBaseURL}/beer -H 'api-key: ${testUserKey}'
    curl -vfk ${publicStagingBaseURL}/beer/Weissbier -H 'api-key: ${testUserKey}'
    curl -vfk ${publicStagingBaseURL}/beer/findByStatus/available -H 'api-key: ${testUserKey}'
    """
  }
```

### 将配置提升到生产网关

每个 3scale 服务都有两个公共 URL:一个名为“staging ”,另一个名为“production”临时 URL 用于在实际应用配置之前测试设置。当配置被升级时，它被自动应用到“生产”网关。

```
  stage("Promote to production") {
   runToolbox([ "3scale", "proxy", "promote", targetInstance,  targetSystemName ])
  }
}

```

## 使用 3scale 工具箱从 Jenkins 管道部署您的 API

恭喜你！您刚刚从 Jenkins 管道部署了您的 API。[完整的管道可以在这里找到，以供参考](https://github.com/3scale/3scale_toolbox/blob/c30042dcd26044325bff8f6a1894bb36d846d570/examples/Jenkinsfile)。

这条管道远非完美；然而，它为在您的管道中使用提供了坚实的基础。在这个博客系列的下一篇文章中，我们将讨论如何增强这个管道的一些缺点:使用 3scale toolbox Jenkins 共享库阅读[。](https://developers.redhat.com/blog/?p=612407)

### 阅读更多

*   [从 CI/CD 管道部署 API 的 5 个原则](https://developers.redhat.com/blog/?p=608917)
*   [3 规模工具箱:从 CLI 部署 API](https://developers.redhat.com/blog/?p=611307)
*   [使用 3scale 工具箱詹金斯共享库](https://developers.redhat.com/blog/?p=612407)

*Last updated: January 14, 2022*
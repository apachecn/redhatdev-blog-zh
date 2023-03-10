# 使用 3scale 工具箱 Jenkins 共享库

> 原文：<https://developers.redhat.com/blog/2019/07/31/using-the-3scale-toolbox-jenkins-shared-library>

在本系列的前一篇文章[从 Jenkins 管道](https://developers.redhat.com/blog/?p=612387)部署您的 API 中，我们发现了 3scale 工具箱如何帮助您在[Red Hat open shift](http://developers.redhat.com/openshift/)/Kubernetes 上从 Jenkins 管道部署您的 API。在本文中，我们将通过使用 3scale toolbox Jenkins 共享库来改进上一篇文章中的管道，使其更健壮、更简洁，并提供更多功能。

## 有哪些需要改进的地方

尽管它并不完美，但我们在上一篇文章中设计的管道是简单且独立的。然而，为了支持生产工作负载，需要改进一些次要方面:

*   “runToolbox”助手方法在每个管道中都被复制。
*   “runToolbox”方法中使用了几个延迟，调整这些延迟被证明是很棘手的。
*   一切都是硬编码的:如果我们需要更改一些 API 元数据，管道代码需要更新。“清单”将代码与配置分开。
*   测试凭证是硬编码的。为了符合大多数公司的安全策略，我们需要动态地生成这些测试凭证。
*   “runToolbox”帮助器方法不是为处理 3scale 工具箱的多个并行运行而设计的。

这些改进并不困难或复杂；这是每个詹金斯管道作家的日常面包！

## 介绍工具箱 Jenkins 共享库

为了帮助 Jenkins Pipeline 作者，围绕 Red Hat Integration 的社区提出了一个名为 [3scale-toolbox-jenkins](https://github.com/rh-integration/3scale-toolbox-jenkins) 的 Jenkins 共享库。它具有以下改进:

*   所有代码都已经在一个 [Jenkins 共享库](https://jenkins.io/doc/book/pipeline/shared-libraries/)中进行了分解，并且可以在您的所有管道中重用。
*   每当管道必须等待动作完成时，就使用轮询循环。没有更多的延迟调谐。
*   工具箱 Jenkins 共享库提供了一个 API 元数据清单。您可以更改 API 元数据，而不必更改管道代码。
*   测试凭证由基于哈希的消息验证码(HMAC)函数动态生成。使用 HMAC 代替随机数据，以保持幂等性。不管 Jenkins 管道运行了多少次，测试凭证保持不变，但仍然不可访问。
*   工具箱的多个并行运行是可能的，因为所有的 Kubernetes 对象都以 Jenkins 构建名称和编号为前缀和标签。
*   它使用下面的 [Jenkins OpenShift 客户端插件](https://github.com/openshift/jenkins-client-plugin)，这使得它比裸露的`oc`命令更可靠。
*   实现语义版本化是为了简化多个 API 版本的管理。

## 使用工具箱 Jenkins 共享库的第一步

在 Jenkins 管道的开始，导入工具箱 Jenkins 共享库:

```
library identifier: '3scale-toolbox-jenkins@master',
        retriever: modernSCM([$class: 'GitSCMSource',
        remote: 'https://github.com/rh-integration/3scale-toolbox-jenkins.git'])
```

声明一个保存 ThreescaleService 对象的全局变量，以便可以在管道的不同阶段使用它:

```
def service = null
```

在 Jenkins 管道的早期阶段，您可以从 API 元数据清单创建 ThreescaleService 对象:

```
service = toolbox.prepareThreescaleService(
    openapi:          [ filename: "swagger.json" ],
    environment:      [ baseSystemName: "my_service" ],
    toolbox:          [ openshiftProject: "toolbox",
                        destination: "3scale-tenant",
                        secretName: "3scale-toolbox" ],
    service:          [:],
    applications:     [ [ name: "my-test-app", 
                          description: "This is used for tests", 
                          plan: "test", 
                          account: "john" ] ],
    applicationPlans: [ [ systemName: "test", 
                          name: "Test", 
                          defaultPlan: true, 
                          published: true ],
                        [ systemName: "silver",
                          name: "Silver" ],
                        [ artefactFile: "https://raw.githubusercontent.com/redhatHameed/API-Lifecycle-Mockup/master/testcase-01/plan.yaml"] ]
)
```

在本例中，API 元数据清单已经内联到管道中，但是您可以将它存储在 Git 存储库中的 YAML 文件中，并使用 [readYAML 步骤](https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/#readyaml-read-yaml-from-files-in-the-workspace-or-text)加载它。这样，您的 API 元数据可以改变，但您的管道代码保持不变。

然后，您可以在管道中的任何相关位置创建所有 API 管理对象:

```
service.importOpenAPI()
echo "Service with system_name ${service.environment.targetSystemName} created !"
service.applyApplicationPlans()
service.applyApplication()
```

运行端到端测试也很容易。请注意测试凭证是如何自动管理的:

```
def proxy = service.readProxy("sandbox")
sh """
curl -vfk ${proxy.sandbox_endpoint}/api/beer -H 'api-key: ${service.applications[0].userkey}'
curl -vfk ${proxy.sandbox_endpoint}/api/beer/Weissbier -H 'api-key: ${service.applications[0].userkey}'
curl -vfk ${proxy.sandbox_endpoint}/api/beer/findByStatus/available -H 'api-key: ${service.applications[0].userkey}'
"""
```

最后，您可以将新配置提升到生产网关:

```
service.promoteToProduction()
```

## Jenkins 管道示例

我们准备了五个 Jenkins 管道，展示了 3scale toolbox Jenkins 共享库在不同环境中的使用:

*   一个非常简单的 API，使用 API 密钥保护，部署在 3 个规模的主机中。
*   混合架构中部署的开放 API(无安全性):3 个规模托管和内部部署。
*   使用 OpenID Connect 保护的 API 部署在相同的混合架构中。
*   相同的 API 部署在三个不同的环境中(开发、测试和生产)
*   在这三个环境中部署了一个应用了语义版本控制的 API(发布了四个版本，结合了不同的安全方案)。

您可以在[3 scale-toolbox-Jenkins-samples repository](https://github.com/rh-integration/3scale-toolbox-jenkins-samples)中找到这些示例。

如果您喜欢真实世界的例子，[integration app-Automation repository](https://github.com/rh-integration/IntegrationApp-Automation)包含一个复合应用程序，展示了通过 Jenkins 管道部署的 API。

## 结论

在本文中，我们为 Jenkins 管道编写者提供了一种使用 3scale 工具箱发布 API 的便捷方式。这个 Jenkins 共享库作为最佳实践和示例代码提供给 Jenkins 管道编写者，供他们在日常工作中使用。您可以选择原样重用这个库并贡献给上游社区，或者复制它并使它成为您的！

### 阅读更多

*   [从 CI/CD 管道部署 API 的 5 个原则](https://developers.redhat.com/blog/?p=608917)
*   [3 规模工具箱:从 CLI 部署 API](https://developers.redhat.com/blog/?p=611307)
*   [从 Jenkins 管道部署您的 API](https://developers.redhat.com/blog/?p=612387)

*Last updated: July 29, 2019*
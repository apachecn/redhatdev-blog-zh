# Kubernetes 和 OpenShift 的自动化测试和度量收集(第 3 部分)

> 原文：<https://developers.redhat.com/blog/2019/01/16/openshift-kubernetes-automated-performance-tests-part-3>

这是基于我在 EMEA 红帽技术交流举办的一次会议的三篇文章系列的第三篇。在[的第一篇文章](https://developers.redhat.com/blog/2018/11/22/automated-performance-testing-kubernetes-openshift/)中，我介绍了利用 [Red Hat OpenShift](https://www.openshift.com) 或 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 进行自动化性能测试的基本原理和方法，并且概述了设置。在第二篇文章的[中，我们看到了构建一个可观察性堆栈。在第三部分中，我们将看到如何自动化性能测试的执行，以及如何收集相关的指标。](https://developers.redhat.com/blog/2019/01/03/leveraging-openshift-or-kubernetes-for-automated-performance-tests-part-2/)

在我的 [GitHub 库](https://github.com/fgiloux/auto-perf-test/)中有一个本文描述的例子。

## 设置概述

[![Overview](img/e7a230144aff03db79f58db5a4382f92.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/img_5bdc69ff84eb6.png)

## 测试自动化

### 测试计划

正如我们在第一篇文章中所看到的，在构建一个代表生产场景并且可以被轻松重用的测试计划时，需要考虑几个方面。让我们浏览一下我的 [GitHub 库](https://github.com/fgiloux/auto-perf-test/tree/master/jmeter/examples)中提供的例子，看看我们如何处理它们。

我的第一个建议是使用 JMeter GUI(本文使用 JMeter 4)及其标准组件进行快速实验，并在设计阶段作为反馈循环。这里是[的一个例子](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/examples/jms-declarative.jmx)。

当我们对设计满意时，我们可以继续用 Groovy(一种用于 Java 平台的动态语言，也与 Java 语法兼容)编写定制组件，这提供了更多的功能和灵活性，但也需要更多的工作。参见[这里的](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/examples/jms-code.jmx)例子。运行 JMeter GUI 就像[下载归档文件](https://jmeter.apache.org/download_jmeter.cgi)，提取它，并调用`./bin/jmeter`一样简单。

让我们使用 JSR 223 采样器(Groovy)来完成 JMS 测试套件。

**总体结构**

测试计划示例由以下内容组成:

*   包含控制器/采样器、定时器和数据集的线程组
*   用户定义的变量
*   监听器:数据写入器和后端监听器

这里不使用断言。

根据 [JMeter 文档](https://jmeter.apache.org/usermanual/test_plan.html)，线程组是任何 JMeter 测试计划的起点。测试计划的所有元素都必须在线程组下定义。还使用了安装和拆卸线程组:它们是特殊形式的线程组，分别用于在常规线程组执行之前和之后执行必要的操作。我们使用它们来加载消息模板和创建到代理的连接。

当我们希望代码在一个线程中只处理一次时，我们使用全局用户定义变量来分配连接字符串参数或定义标志。

关于监听器，数据写入器和后端监听器提供对 JMeter 收集的关于测试用例的信息的访问，并允许它被记录在文件中(参见[简单数据写入器](https://jmeter.apache.org/usermanual/component_reference.html#Simple_Data_Writer))或 InfluxDB 中(参见[后端监听器](https://jmeter.apache.org/usermanual/component_reference.html#Backend_Listener))。

**配置具体化**

有两种方法可以将属性传递给 JMeter 测试计划。首先是在`./bin/user.properties`中定义它们。第二种是在启动时使用`-J`传递它们，例如`-JBROKER`。我们将看到当 JMeter 在 [OpenShift](http://openshift.com/) 上的[容器](https://developers.redhat.com/blog/category/containers/)中运行时，如何通过一个简单的启动脚本将注入的环境变量作为属性传递，从而轻松利用后一种形式。

然后可以在用户定义的变量中使用属性。这就是我们正在使用的，例如，AMQP 连接字符串:

```
${__P(BROKER,messaging-perftest.apps.sandbox.com)}amqps://${__P(BROKER,messaging-perftest.apps.sandbox.com)}:${__P(PORT,443)}?transport.trustStoreLocation=${__P(JKS_LOCATION,/myrepolocation/auto-perf-test/camel-amq-fakeapp/src/main/resources/amqp-certs/amqp.jks)}&transport.trustStorePassword=${__P(JKS_PWD,redhat)}&transport.verifyHost=false
```

在本例中，如果没有传递名为`BROKER`的参数，则应用默认值`messaging-perftest.apps.sandbox.com`。然后可以在 Groovy 中用`vars.get("connection_string")`检索变量，用`props.get("PAIN_TEMPLATE_LOCATION")`检索属性。在声明性组件中，可以使用`${connection_string}`(在我的 GitHub 存储库中提供的示例中，调用 JMS Purge 元素)。

同样的策略也适用于证书配置，其中 Java 密钥库(JKS)位置和密码作为属性传递。JKS 本身通过挂载一个包含它的秘密被注入到容器文件系统中。

此外，像消息数量、注入速率、持续时间等测试参数可以通过属性注入，这样不同的测试用例可以一个接一个地运行，而无需人工干预或复制测试代码。

**数据集**

正如本系列第 1 部分所述，我们应该使用代表生产数据的数据集。应该可以在两次运行之间重用它们。JMeter 通过 [CSV 数据集配置](https://jmeter.apache.org/usermanual/component_reference.html#CSV_Data_Set_Config)提供了一种便捷的方式。确实有可能提供一个带有像[这样标题的 CSV 文件。使用标题名称创建变量，并用原始值填充变量。CSV 文件的下一行由下一次迭代使用。这与](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/examples/pain_samples.csv) [Groovy 模板功能](http://groovy-lang.org/templating.html)相结合，是创建反映生产数据多样性和不同模式出现的工作负载的强大工具。一个简单的 [XML 模板](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/examples/pain_template.xml)被用作演示的一部分。

**附加库**

经常需要向 JMeter 提供库，以便测试计划可以向代理发送消息，或者与 Jaeger 之类的外部系统集成。因此，只需要将所需的库添加到 JMeter `lib`目录中。

**首先清洗**

我们之前看到，在运行测试之前，需要一个干净的环境。因此，测试计划会清除队列。使用具有不同偏移量(启动延迟)的线程组，可以让清除操作在消息发送到代理之前完成。这是在测试计划中使用 JMS 点对点组件配置的。

**喷射模式**

在第一部分中，我们看到延迟以及某种程度上的吞吐量受到消息注入方式的影响。JMeter 允许使用各种定时器组件重新创建复杂的注入模式。在演示中，我们配置了一个恒定吞吐量计时器，目标是每分钟发送 600 条消息。这适用于当前线程组中的所有活动线程，但 JMeter 允许其他选项，并提供开箱即用的各种计时器。如果都不合适，也可以用 Groovy 编写一个。

### 扩展可观测性

测量点对于获得真实情况和反映生产中应用程序预期的测试结果非常重要。对于代理消息，消息花费的排队时间通常比应用程序处理所需的时间更重要。

为了获得足够接近端到端经过时间的值，可以使用 JMS 时间戳。这就是演示中所做的。发送方设置消息准备发送的时间，接收方从应用程序内部消息发布者设置的 JMS 时间戳中减去该时间。这仍然是一个近似值，但通常不会超过几毫秒。这允许脱离“瞬时”读取的要求。

JMeter 收集的所有结果都可以保存在一个文件中。这是演示中简单的数据编写器所做的工作。JMeter 还能够将结果导出到 InfluxDB。后端监听器开箱即用(如演示中所示)。为此，也可以使用 Groovy 脚本，这将在数据选择方面提供额外的灵活性。

InfluxDB 中的 JMeter 数据可以通过 Grafana 检索和显示。这就是在本系列的第二部分中介绍的仪表板中所做的事情。这是 JMeter 图。

[![JMeter graphs](img/8cac4b2bbe79584b474243457525daa4.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/jmeter-grafana.png)

以类似的方式，我们在上一篇文章中看到的跟踪范围可以扩展到包括 JMeter 发送者和接收者。对于这个演示，OpenTracing 和 Jaeger 库已经被添加到了`lib`目录中。在设置线程组中为每个线程创建一个跟踪器，并将其添加到属性中。然后可以在发送者代码中检索它，这里使用了一个`TracingMessageProducer`来代替标准的 JMS 生成器。类似地，跟踪器使用了一个`TracingMessageConsumer`,而不是消费者代码中的标准 JMS 组件。这在 Jaeger 中产生了我们在上一篇文章中已经看到的结果。

[![Result in Jaeger](img/620bc6fe81e3a2107a8084acfa2c4fe2.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/jaeger.png)

### 集装箱化

在容器中运行 JMeter 的第一步是用它和我们添加的库创建一个映像。在演示中，这是通过[这个 Dockerfile](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/container/Dockerfile) 来完成的。

另一个方面是参数需要在启动时提供给 JMeter 进程。启动脚本会这么做。该逻辑基于以下约定:每个以`J_`为前缀的环境变量都被传递给 JMeter 进程。

该脚本还运行特定目录中提供的测试计划。这允许我们将一个 [configMap](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/openshift/apt-jmx-cm.yaml) 挂载到容器文件系统中，并执行由它提供的测试计划以及所需的数据集和模板，而无需重新创建容器映像。类似地，用于与代理通信的证书通过 secrets 注入，并安装到容器文件系统中。测试结果存储在一个持久卷上，这使得将同一个目录装载到 Jenkins 容器中成为可能。

最后，如果已经提供了一个 URL 来通知 Jenkins(它也可以与其他 CI 工具一起工作)测试计划的执行已经完成，那么这个脚本就会发出一个 REST 调用。

最后一点是，边车容器用于接管与 Jaeger 服务器的通信。这可以在[部署配置](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/openshift/apt-jmeter-job-persistent-tm.yaml)中看到。

关于如何构建容器映像并在 OpenShift 上运行它的详细说明可以在 [my GitHub repository](https://github.com/fgiloux/auto-perf-test/tree/master/jmeter) 中找到。

## 管弦乐编曲

最后缺少的一点是，我们如何自动化整个链:从用最新的代码版本重新创建一个干净的环境到触发测试运行、断言结果、标记代码、执行配置、创建容器映像并最终将它们提升到下一个环境？

### 管道

Jenkins 是 [Kubernetes](https://developers.redhat.com/blog/category/kubernetes/) 上 [CI/CD](https://developers.redhat.com/blog/category/ci-cd/) 的首选工具，它与 OpenShift 一起提供。[管道](https://jenkins.io/doc/book/pipeline/)是自动化这些过程的推荐方法。当你开始为越来越多的组件使用 Jenkins 和 pipelines 时，你会发现它们之间有很多相似之处。为了重用，最好使用[共享库](https://jenkins.io/doc/book/pipeline/shared-libraries/)。在我们的演示中没有这样做，但这是留给读者的一个简单任务。

现在让我们看看[演示管道](https://github.com/fgiloux/auto-perf-test/blob/master/jenkins/pipelines/Jenkinsfile)的步骤。

**触发**

理想情况下，性能测试应该在每次提交到主干之后运行。它们确实应该是非回归测试的一部分。有一些插件可以集成各种版本控制系统(VCS ),比如这个版本控制系统(T1)。然而，由于资源要求高或测试时间长，这可能是不可能的。在这种情况下，按时运行是下一个最好的事情。这可以通过我们的 Jenkins 文件中的下面一行代码来完成，它将每天执行一次流水线，并自动分发(您不希望所有的测试都在完全相同的时间被触发)。

```
cron('H H * * *')
```

**与 OpenShift 集群交互**

与 OpenShift 交互的必要插件是在默认的 Jenkins 容器中配置好的。许多操作通过 [OpenShift Jenkins 管道插件](https://github.com/openshift/jenkins-client-plugin)变得简单，例如，显示当前项目的名称:

```
openshift.withCluster() {
  openshift.withProject() {
    echo "Using project: ${openshift.project()}"
  }
}
```

**清洁纸张**

当我们运行我们的测试计划时，我们想要确保我们正在测试期望的代码版本，并且没有由于先前测试运行的剩余部分而产生的影响。因此，“删除并重新创建”是最好的方法。这可以在项目/名称空间级别完成，或者标签可以用于细粒度方法。将一个公共标签应用到作为测试运行的一部分部署的所有对象是很容易的。然后可以使用下面几行来检索和删除它们，其中`testPlanName`特定于管道或管道运行，如果我们希望允许并行运行的话。

```
if (openshift.selector("all", [ "testplan" : testPlanName ]).exists()) {
  openshift.selector("all", [ "testplan" : testPlanName ]).delete()
}
// all not being all
if (openshift.selector("secrets", [ "testplan" : testPlanName ]).exists()) {
    openshift.selector("secrets", [ "testplan" : testPlanName ]).delete()
}
if (openshift.selector("configMaps", [ "testplan" : testPlanName ]).exists()) {
    openshift.selector("configMaps", [ "testplan" : testPlanName ]).delete()
}
```

**供应**

模板可用于创建所需的对象:

```
openshift.withCluster() {
  openshift.withProject() {
    // "template" is the name of the template loaded in OpenShift
    openshift.newApp(template)
  }
}
```

也可以创建单个对象:

```
def testPlanPath = 'https://raw.githubusercontent.com/fgiloux/auto-perf-test/master/jmeter/openshift/apt-jmx-cm.yaml'
def testPlanCm = openshift.create(testPlanPath).object()
// Applying label makes it is easy to recognise for cleansing
testPlanCm.metadata.labels['testplan'] = testPlanName
openshift.apply(testPlanCm)
```

然而，在现实生活中，您可能希望将完整的配置存储在 VCS 中，并通过构建管道从克隆的存储库中进行检查和创建(与上面在目录中可用的配置文件的循环中执行的代码相同)。

**构建应用映像**

对此，有两派观点。最简单的方法是用 S2I 在 OpenShift 中完成构建。在这种情况下，您只需使用在环境供应期间创建的构建配置开始构建(可能将 repo 分支/标记作为参数传递),剩下的工作由 OpenShift 完成:

```
def builds = openshift.selector("bc", 'camel-amq-fakeapp-s2i').startBuild("--wait=true")
```

另一种方法是使用 Maven slave(在 OpenShift 上的容器中运行)来构建工件，将它们推送到工件存储库(如 Nexus 或 Artifactory)，通过 SonarQube 对它们进行扫描，然后发布(更新版本号)并用于构建最终的容器映像，其中源代码、映像和工件都有类似的标记。然而，描述这一点超出了本文的范围。

**推出应用**

既然已经创建了映像，我们可以在我们的测试环境中运行应用程序，并等到它启动后再启动测试:

```
def dc = openshift.selector("dc", 'camel-amq-fakeapp').rollout()
timeout(5) {
    openshift.selector("dc", 'camel-amq-fakeapp').related('pods').untilEach(1) {
        return (it.object().status.phase == "Running")
}
```

**测试计划执行**

我们现在已经走得够远了，可以进行测试了。我们可能首先想要注册一个 [webhook](https://wiki.jenkins.io/display/JENKINS/Webhook+Step+Plugin) ，它被传递给我们之前看到的运行 JMeter 的脚本:

```
def hook = registerWebhook()
def callbackUrl = hook.getURL()
```

下一步是使用所需的参数运行 JMeter(最好作为[作业](https://github.com/fgiloux/auto-perf-test/blob/master/jmeter/openshift/apt-jmeter-job-tm.yaml))，其中`BUILD_NUMBER`由 Jenkins 自动生成并标识运行:

```
def models = openshift.process("apt-jmeter-job", "-p", "JMX_CONFIGMAP=${testPlanName}","-p","RESULT_SUB_DIR=${JOB_NAME}/${BUILD_NUMBER}","-p","CALLBACK_URL=${callbackUrl}")
```

Jenkins 插件让我们可以轻松地操作对象定义，例如，为我们的作业添加标签:

```
for ( o in models ) {
  o.metadata.labels['testplan'] = testPlanName
}
```

结果可以应用于 OpenShift:

```
def created = openshift.create(models)
```

此时，JMeter 容器正在执行测试计划。我们可以等到它完成:

```
timeout(10) {
  print "Waiting for tests to complete..."
  waitForWebhook hook
}
```

正如我们前面看到的，测试结果可以在 Jenkins 安装的共享驱动器上获得，所以我们现在可以让 Jenkins [性能插件](https://wiki.jenkins.io/display/JENKINS/Performance+Plugin)处理它们:

```
perfReport sourceDataFiles: "/opt/performances/${JOB_NAME}/${BUILD_NUMBER}/*.jtl", compareBuildPrevious: true, modePerformancePerTestCase: true, modeOfThreshold: true, relativeFailedThresholdPositive: 50, relativeUnstableThresholdNegative: 40, relativeUnstableThresholdPositive: 40
// Job delete required due to the jaeger-agent sidecar not terminating with JMeter
openshift.selector('job', ['testplan': testPlanName]).delete()
```

这个插件提供了一些图表来快速概述测试是如何执行的。

[![Graphs that show performance information](img/767cd1fe2dc2f06bb21458876d85185d.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/jenkins-jmeter-overview.png)


更重要的是，该插件提供了根据先前运行的阈值或偏差来调节管道结果和下一阶段(通过或失败)的可能性。

**标记**

最后，我们可能希望根据结果来标记我们的图像:

```
openshift.tag("camel-amq-fakeapp:latest", "camel-amq-fakeapp:staging")
```

### 图像扩展

正如我们所看到的，我们可能会使用额外的插件。在离线环境中，需要将它们添加到 Jenkins 映像中。在 OpenShift 中，这是通过 [S2I 过程](https://docs.openshift.com/container-platform/3.10/using_images/other_images/jenkins.html#jenkins-as-s2i-builder)直接完成的。标准的 Jenkins 模板也进行了修改，让 Jenkins 容器装载带有测试结果的持久卷。这是可用的[这里](https://github.com/fgiloux/auto-perf-test/blob/master/jenkins/openshift/perftest-jenkins-persistent-tm.yaml)。

## 结论

咻；这是一个相当长的系列！恭喜坚持到最后的读者。我的最后一句话是要感谢我的同事 Shrish Srivastava，他在普罗米修斯和格拉法纳事件中帮助了我。

## “利用 OpenShift 或 Kubernetes 进行自动化性能测试”系列中的所有文章

*   [第 1 部分:基本原理和方法](https://developers.redhat.com/blog/2018/11/22/automated-performance-testing-kubernetes-openshift/)
*   [第 2 部分:构建一个可观测性堆栈](https://developers.redhat.com/blog/2019/01/03/leveraging-openshift-or-kubernetes-for-automated-performance-tests-part-2/)
*   第 3 部分:自动化测试和度量收集(本文)

*Last updated: September 3, 2019*
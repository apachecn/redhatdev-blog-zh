# 加速 Tekton 管道中的 Maven 构建

> 原文：<https://developers.redhat.com/blog/2020/02/26/speed-up-maven-builds-in-tekton-pipelines>

[Tekton](https://tekton.dev) 是一个开源项目，提供标准的 Kubernetes 风格的资源和构建模块，用于创建可以在任何 Kubernetes 上运行的 CI/CD 管道。Tekton 通过引入许多自定义资源定义(CRD)来实现这一点，例如`Pipeline`、`Task`和`ClusterTask`，以提供用于定义交付管道的语言和结构，如图 1 所示。Tekton 还提供了一组控制器，每当用户创建上述资源时，这些控制器负责按需运行 pod 中的管道。

[![Diagram of a Pipeline containing a Task workflow.](img/995cfa986e2586b7ba0d6bf807d744e9.png "Tekton Pipeline")](/sites/default/files/blog/2020/02/image2.png)

图 1:tek ton 管道包含一系列任务。">

Tekton 的使用在过去一年中迅速增长。一个经常被请求的特性是在任务之间共享工件的能力，以便为 Maven 和 NPM 等构建工具缓存依赖关系。尽管以前可以在任务中使用卷，但 Tekton 0.10 的发布增加了对工作区的支持，这使得管道中的任务更容易使用持久卷来共享工件。

在本文中，我们将探讨如何使用工作区来缓存 Java 构建中的 Maven 依赖项，以便消除为每个构建下载依赖项的需要。

## Tekton 工作区

*tek ton Pipelines 中的 Workspaces* 是指一个管道在运行时需要的共享卷的声明。它们类似于卷，只是不提供实际的卷，只声明意图。在管道定义中，工作区可以作为共享卷传递给相关任务。结果是，当向许多任务提供相同的工作空间时，它们都可以从完全相同的卷中读取和写入，并根据需要共享文件和工件。

值得一提的是，虽然卷指的是用于缓存 Maven 依赖项的持久卷，但它也可以是一个`ConfigMap`，或者是传递给管道运行以在任务之间安装和共享的秘密。

让我们看看如何在实践中使用工作区来缓存 Maven 依赖项。

## 带工作区的 Maven 任务

为了在管道中构建 Maven 项目，应该定义一个 Maven 任务。Tekton 目录已经包含了一个 [Maven 任务](https://github.com/tektoncd/catalog/tree/master/maven)。然而，我们需要这个任务的一个修改版本来为 Maven 的依赖项声明一个工作空间:

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: mvn
spec:
  workspaces:
  - name: maven-repo
  inputs:
    params:
    - name: GOALS
      description: The Maven goals to run
      type: array
      default: ["package"]
    resources:
    - name: source
      type: git
  steps:
    - name: mvn
      image: gcr.io/cloud-builders/mvn
      workingDir: /workspace/source
      command: ["/usr/bin/mvn"]
      args:
        - -Dmaven.repo.local=$(workspaces.maven-repo.path)
        - "$(inputs.params.GOALS)"

```

这个任务与 Tekton 目录中的任务非常相似，不同之处在于定义了一个名为`maven-repo`的工作空间。该工作区规定，无论何时运行该任务，都应该提供并安装一个卷作为本地 Maven 存储库。然后，这个工作区的路径被传递给 Maven 命令，以便与`-Dmaven.repo.local=$(workspaces.maven-repo.path)`一起用作本地 Maven 存储库。

可以配置安装工作空间的路径；但是，在本例中，默认的挂载路径就足够了。

## 构建带有工作空间的管道

现在，让我们定义一个使用 Maven 任务构建 Java 应用程序的管道。为了演示 Maven 依赖项的缓存效果，下面的管道(如图 2 所示)运行三个 Maven 任务来执行构建、集成任务，并为测试结果、代码覆盖率等生成报告。

[![Diagram showing the Maven build pipeline's workflow](img/33ed458ca4ef1739b58027d68dfd9145.png "Maven Pipeline")](/sites/default/files/blog/2020/02/image4.png)

图 2:一个示例 Tekton Maven 构建管道。">

图 2 中代表管道的管道定义是:

```
apiVersion: tekton.dev/v1alpha1
kind: Pipeline
metadata:
  name: maven-build
spec:
  workspaces:
  - name: local-maven-repo
  resources:
  - name: app-git
    type: git
  tasks:
  - name: build
    taskRef:
      name: mvn
    resources:
      inputs:
      - name: source
        resource: app-git
    params:
    - name: GOALS
      value: ["package"]
    workspaces:
    - name: maven-repo
      workspace: local-maven-repo
  - name: int-test
    taskRef:
      name: mvn
    runAfter: ["build"]
    resources:
      inputs:
      - name: source
        resource: app-git
    params:
    - name: GOALS
      value: ["verify"]
    workspaces:
    - name: maven-repo
      workspace: local-maven-repo
  - name: gen-report
    taskRef:
      name: mvn
    runAfter: ["build"]
    resources:
      inputs:
      - name: source
        resource: app-git
    params:
    - name: GOALS
      value: ["site"] 
    workspaces:
    - name: maven-repo
      workspace: local-maven-repo

```

注意管道的`local-maven-repo`工作空间的声明。它指出，当这个管道要运行时，应该提供一个卷并将其用作这个工作空间。然后，这个工作空间被提供给这个管道中的每个任务，以便它们都共享同一个工作空间。

## 运行 Maven 管道

现在可以运行管道来构建一个 Java 应用程序，如 [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) 示例应用程序。在启动管道之前，需要一个 [`PersistentVolumeClaim` (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 来提供一个工作空间来缓存 Maven 依赖项:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: maven-repo-pvc
spec:
  resources:
    requests:
      storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain 

```

现在，您可以创建一个使用上述 PVC 作为管道工作空间的管道运行:

```
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  generateName: petclinic-run-
spec:
  pipelineRef:
    name: maven-build
  resources:
  - name: app-git
    resourceSpec:
      type: git
      params:
        - name: url
          value: https://github.com/spring-projects/spring-petclinic
  workspaces:
  - name: local-maven-repo
    persistentVolumeClaim:
      claimName: maven-repo-pvc

```

注意`maven-repo-pvc` PVC 和为缓存 maven 依赖项而声明的工作空间之间的映射。因此，这个 PVC 被传递给管道和相应的任务，作为缓存文件和工件的共享卷。

由于这是 Maven 目标第一次运行，管道运行将需要时间来下载依赖项并完成执行:

```
$ tkn pr list
NAME                  STARTED          DURATION     STATUS
petclinic-run-6l5w7   16 minutes ago   9 minutes    Succeeded

```

您可以在这里看到，在我的环境中，管道运行花了大约 9 分钟才完成。您还可以获得每个任务执行所用时间的明细(参见图 3):

```
$ tkn pr describe petclinic-run-6l5w7
...
 Taskruns

 NAME                                     TASK NAME    STARTED          DURATION    STATUS
 ∙ petclinic-run-6l5w7-gen-report-s6mhf   gen-report   16 minutes ago   4 minutes   Succeeded
 ∙ petclinic-run-6l5w7-int-test-8tbkn     int-test     16 minutes ago   2 minutes   Succeeded
 ∙ petclinic-run-6l5w7-build-4gg4l        build        21 minutes ago   4 minutes   Succeeded

```

[![Red Hat OpenShift Container Platform -&gt; Developer -&gt; Pipelines screenshot](img/2be24d9672719c42269a3d2855866feb.png "OpenShift Console")](/sites/default/files/blog/2020/02/image3.png)

图 3:在 Red Hat OpenShift 容器平台中查看管道运行结果。">

通过再次使用`kubectl create`YAML 再次运行管道，并观察执行时间:

```
$ tkn pr list
NAME                  STARTED          DURATION     STATUS
petclinic-run-qb64z   7 minutes ago    4 minutes    Succeeded
petclinic-run-6l5w7   40 minutes ago   9 minutes    Succeeded

```

请注意，在我的环境中，执行时间显著减少到了大约四分钟。任务执行时间的细分可以更准确地显示效果:

```
$ tkn pr describe petclinic-run-qb64z
...

 Taskruns

 NAME                                     TASK NAME    STARTED         DURATION    STATUS
 ∙ petclinic-run-qb64z-int-test-ppwgc     int-test     4 minutes ago   2 minutes   Succeeded
 ∙ petclinic-run-qb64z-gen-report-mhhmj   gen-report   4 minutes ago   2 minutes   Succeeded
 ∙ petclinic-run-qb64z-build-ck7cp        build        5 minutes ago   1 minute    Succeeded

```

测试任务运行没有受到太大影响，因为它使用了构建任务运行中下载的大部分依赖项，甚至在第一次管道运行中也是如此，如图 4 和图 5 所示。

[![Red Hat OpenShift Container Platform -&gt; Developer -&gt; Pipelines -&gt; Pipeline Details](img/1553a3ac568dbbbe4976e262b32013cc.png "OpenShift Console")](/sites/default/files/blog/2020/02/image1.png)

图 4:比较两次运行的管道细节。">

[![Red Hat OpenShift Container Platform -&gt; Developer -&gt; Pipelines -&gt; Pipeline Runs -&gt; Pipeline Run Details](img/ac0dfff24e10e36cbfce8bfc9139e677.png "OpenShift Console")](/sites/default/files/blog/2020/02/image5.png)

图 5:查看管道运行概述。">

## 结论

Tekton 0.10 中的工作空间支持简化了管道中任务之间的文件和工件共享，例如将 JAR 文件从一个任务传递到另一个任务，或者缓存构建依赖关系，如本文中所示。然而，这仅仅是 Tekton 所能做的事情的开始，Tekton 社区正在通过为 TektonCD CLI 提供支持来改进工作区用户体验。

## 尝试一下

本文中使用的所有文件都可以在下面的 GitHub 资源库中找到:
【https://github.com/siamaksade/tekton-pipelines-maven-demo】T2。要使用它们，在您的工作站上下载并安装 [CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview) 和 [TektonCD CLI](https://github.com/tektoncd/cli/releases) ，然后从`canary` operator channel 安装[open shift Pipelines Operator](https://openshift.github.io/pipelines-docs/docs/0.8/proc_installing-pipelines-operator-in-web-console.html)，在平台上启用 Tekton Pipelines 0.10。

此后，运行以下命令来创建管道:

```
$ oc create -f pvc.yaml
$ oc create -f maven-task.yaml
$ oc create -f maven-pipeline.yaml
$ oc create -f maven-pipelinerun.yaml

```

*Last updated: April 7, 2022*
# Red Hat OpenShift 上的 Tekton 入门

> 原文：<https://developers.redhat.com/blog/2019/07/19/getting-started-with-tekton-on-red-hat-openshift>

我最近听说 Tekton 作为 Jenkins 在 Red Hat OpenShift 的替代者。引起我注意的是 Tekton 使用操作符作为构建模块，操作符也是我感兴趣的东西。不过，我不想超越自己；所以我们先从在红帽 OpenShift 上安装 Tekton 开始。在 Kubernetes 上安装也是可能的，但目前的重点是 OpenShift。

要安装 Tekton，您需要是 Red Hat OpenShift 集群的集群管理员。原因是控制器必须与`anyuid`一起运行。只需在手边准备一个 OpenShift 或 Minishift 集群，您就可以对其进行集群管理。

我们将使用一个专用项目在以下位置安装 Tekton 操作器:

```
oc new-project tekton-pipelines --display-name='Tekton Pipelines'
oc adm policy add-scc-to-user anyuid -z tekton-pipelines-controller
oc apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml
```

这在 tekton-pipelines 项目中创建了两个部署，命名为`tekton-pipelines-controller`和`tekton-pipelines-webhook`。启动速度很快，但要查看吊舱是否正在运行，请使用以下`oc`命令并等待“运行”状态:

```
oc get pods --namespace tekton-pipelines --watch
```

使用 CTRL + C 退出监视模式。

现在 Tekton 正在运行，我们需要定义一个包含步骤的任务，说明要做什么。我们还需要一个 TaskRun 定义来说明运行哪个任务。为此，我们需要两个 yaml 文件。这些文件如何命名并不重要，但是我使用惯例用`-task.yaml`和`-task-run.yaml`来结束它们。

名为`echo-hello-world-task.yaml`的任务定义文件:

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: echo-hello-world-task
spec:
  steps:
    - name: echo
      image: ubuntu
      command:
        - echo
      args:
        - "hello world"

```

以及一个名为`echo-hello-world-task-run.yaml`的 TaskRun 定义文件:

```
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: echo-hello-world-task-run
spec:
  taskRef:
    name: echo-hello-world-task

```

我们将使用以下命令应用这两个文件:

```
oc apply -f echo-hello-world-task.yaml
oc apply -f echo-hello-world-task-run.yaml

```

一旦应用了这两个文件，它们将被直接执行。要跟踪 TaskRun 的输出，请使用以下命令:

```
oc get taskruns/echo-hello-world-task-run -o yaml

```

查看输出并搜索:

```
status:
  conditions:
    - lastTransitionTime: 2019-07-08T18:48:15Z
      status: "True"
      type: Succeeded

```

如果它看起来几乎和上面一样，那么您已经执行了将成为完整 Tekton 管道的第一部分。

*Last updated: September 10, 2020*
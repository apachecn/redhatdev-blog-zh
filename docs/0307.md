# 改编 Docker 和 Kubernetes 容器以在 Red Hat OpenShift 容器平台上运行

> 原文：<https://developers.redhat.com/blog/2020/10/26/adapting-docker-and-kubernetes-containers-to-run-on-red-hat-openshift-container-platform>

越来越多的公司将他们的应用程序迁移到 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/getting-started) (RHOCP)。这个企业级容器平台是安全和全面的，基于行业标准，包括与 Docker 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 相关的标准。然而，由于更严格的安全限制，在 Docker 和 Kubernetes 上运行的容器在没有修改的情况下可能无法在 Red Hat OpenShift 上成功运行。

[Red Hat OpenShift Container Platform](https://developers.redhat.com/products/openshift/getting-started)是一个完全托管的 Red Hat open shift 服务，它利用了企业级的可伸缩性和安全性。它直接与 Kubernetes 集成，并为应用程序部署提供了几种模型。例如，由于容器引擎中的安全漏洞，OpenShift 可以降低在容器中运行的进程在主机上被授予升级权限的风险。出于这个原因，容器使用一个任意分配的用户 ID 运行*。*

相反，在 Docker 和 Kubernetes 中，容器要么以 Docker 文件中用户指令指定的用户身份运行，要么以 root 用户身份运行(如果没有指定用户指令)。设计为以根用户身份运行的容器化应用程序在 OpenShift 上可能无法按预期运行。

本文回顾了我在改编 Docker 和 Kubernetes 的容器以在 Red Hat OpenShift 上运行时发现的常见问题。首先，我描述了需要解决的潜在问题，这样容器就可以安全地运行在 OpenShift 上，而不需要强制一个非限制性的[安全上下文约束](https://docs.openshift.com/container-platform/4.5/authentication/managing-security-context-constraints.html) (SCC)。然后，我提供了如何创建无需修改就可以在 Kubernetes 和 OpenShift 上运行的图像的技巧。此外，我还提供了调试不能按预期运行的应用程序问题的技术。

## 组所有权和文件权限

尽管 OpenShift 使用任意分配的用户 ID 运行容器，但是组 ID 必须始终设置为根组(0)。因此，在映像中运行的进程需要访问的目录和文件应该将它们的组所有权设置为根组。按照 [OpenShift 容器平台特定指南](https://docs.openshift.com/container-platform/4.5/openshift_images/create-images.html#images-create-guide-openshift_create-images)的建议，它们还需要被该组读取/写入。

将以下内容添加到 Dockerfile 中，将设置目录和文件权限，以允许 root 组中的用户使用与目录和文件所有者相同的授权来访问它们:

```
RUN chgrp -R 0 /some/directory && \
    chmod -R g=u /some/directory
```

## 运行时用户与 Kubernetes 的兼容性

对于这一步，我建议您将 Kubernetes 的运行时用户设置为一个*非根*用户，以便向后兼容。您可以通过将以下内容添加到 Dockerfile 文件，然后相应地更新文件和目录权限来执行此操作:

```
USER 1001
RUN chown -R 1001:0 /some/directory
```

结果，当图像在 OpenShift 上运行时，指定的用户被忽略，因为用户被设置为任意 ID。相比之下，当图像在 Kubernetes 上运行时，许多 OpenShift 限制都会生效，因为容器是作为非根用户运行的。

干得好。运行时用户兼容性有助于确保单个 docker 文件可以用于创建在 OpenShift 和 Kubernetes 上都能正常工作的映像。

## 可执行权限

当容器在 Kubernetes 上作为根用户运行时，文件权限被忽略。相反，当在 OpenShift 上使用任意用户 ID 时，设置权限位以便执行文件。

例如，添加以下内容来设置所有者和组的执行权限位:

```
RUN chmod 775 /some/directory/script
```

## 卷装载

在 OpenShift 中，卷装载由用户/组 root:root 所有，并且每个卷装载都分配有以下权限:

```
drwxrwx---
```

注意，Linux 命令，比如 `chown(1)`、`chgrp(1)`、`chmod(1)`，不能在卷挂载点本身上执行。但是，您可以在卷装载中创建文件或目录，作为具有完全访问权限的根组。

## 特许港口

低于 1024 的 TCP/IP 端口号是特权端口号，仅允许根用户绑定到这些端口。在 OpenShift 上运行容器时，服务器应用程序需要分配大于 1023 的端口号。

## 需要用户名的应用程序

当您试图查找当前运行的用户 ID 的用户名时，应用程序有时会在 OpenShift 上失败。当没有任意分配的用户 ID 的`/etc/passwd`条目时，就会出现这个问题。这个问题的解决方法是在容器启动时为任意分配的用户 ID 添加一个`/etc/passwd`条目。

出于演示目的，在 docker 文件所在的目录中，您需要创建一个名为`uid_entrypoint`的文件，其内容如下。

记得用您选择的用户名替换`myuser`:

```
#!/bin/sh
if ! whoami &> /dev/null; then
  if [ -w /etc/passwd ]; then
    echo "myuser:x:$(id -u):0:My User:${HOME}:/sbin/nologin" >> /etc/passwd
  fi
fi
exec "$@"
```

在用容器中运行的主脚本或程序替换`runcmd`之后，将以下内容添加到 docker 文件中:

```
COPY uid_entrypoint /
RUN chmod g=u /etc/passwd && chmod 775 /uid_entrypoint
ENTRYPOINT ["uid_entrypoint"]
CMD ["runcmd"]
```

注意，当容器开始运行时，您需要为指定的用户名添加一个密码条目(如果它还不存在)，然后主进程开始。

## 部署

由于 Kubernetes 和 OpenShift 之间的不同行为，部署到 OpenShift 有时会失败。

### InitContainers 无助于解决权限问题

在 Kubernetes 中运行时，`initContainers`有时用于设置 pod 中其他容器使用的文件和目录的权限。这个优势依赖于 Kubernetes 作为根用户运行`initContainers`，并作为 Docker 指令`USER`中指定的用户运行其他容器。

当在 OpenShift 上运行时，`initContainers`和常规容器都使用 OpenShift 分配的用户 ID。因此，`initContainers`中的权限与同一个 pod 中运行的常规容器中的权限完全相同。

### SecurityContext 指令

重要的是，因为 OpenShift 分配了一个任意的用户 ID 和一个组 ID`zero (0)`，所以当您在 OpenShift 上运行时，SecurityContext 指令，如`runAsUser`和`runAsGroup`，不能出现在部署规范(或舵图)中。启用 SecurityContext 指令会导致部署失败。

### 避免 OpenShift 项目默认值

在 OpenShift 项目中运行的应用程序 *default* 接收的权限类似于在 Kubernetes 上运行时使用的权限。OpenShift 安全限制不适用于此项目。因此，不要使用这个名称空间来测试容器。

另外，[我建议您不要使用](https://docs.openshift.com/container-platform/4.5/authentication/managing-security-context-constraints.html#role-based-access-to-ssc_configuring-internal-oauth)以下 OpenShift 名称空间来运行 pod 或服务:`default`、`kube-system`、`kube-public`、`openshift-node`、`openshift-infra`、`openshift`。

## 如何调试问题

将映像从 Docker 或 Kubernetes 迁移到 OpenShift 时，映像可能无法开箱即用。因此，我建议您在调试时使用以下工具和方法来找到错误的根本原因:

*   通过运行:

    ```
    $ oc get events -w
    ```

    检查当前命名空间的系统事件中记录的错误
*   通过运行:

    ```
    $ oc logs -f *<podname>* --all-containers
    ```

    检查 pod 日志中的错误
*   使用命令

    ```
    $ oc rsh *<podname>*
    ```

    登录 pod 中的容器，检查文件和权限以及其他问题
*   When the container keeps starting and crashing, in the Dockerfile set:

    ```
    ENTRYPOINT ["sleep", "100000000"]
    ```

    重新运行 pod 并登录，以确定出现问题的原因。

*   在容器中安装`strace(1)`命令，跟踪正在运行的程序的系统调用，看看哪个失败了。

## 结论

在本文中，您了解了如何查看将 Docker 和 Kubernetes 中的容器应用到 OpenShift 时发现的常见问题。我还演示了如何轻松解决这些问题，并创建可以在 Docker、Kubernetes 和 OpenShift 上运行的图像。

*Last updated: April 7, 2022*
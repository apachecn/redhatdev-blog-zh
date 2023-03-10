# 在 OpenShift 上运行 NuGet 服务器

> 原文：<https://developers.redhat.com/blog/2017/10/11/running-nuget-server-openshift>

当你建立你的。默认情况下，NuGet 包是从 NuGet 获取的。但是，有时您可能希望使用本地 NuGet 存储库。例如，您可能想要:

*   使用私有的 NuGet 包，但是你不希望除了你的同事之外的任何人看到它们。
*   在构建服务器附近的服务器上缓存 NuGet 存储库
*   让您的构建服务器与互联网断开连接。

我将解释如何在 OpenShift 上设置一个私有的 NuGet 服务器，以及如何在构建您的。NET Core 项目在 OpenShift 中使用 s2i-dotnetcore。

## 将 nexus 服务器设置为专用 NuGet 服务器

不幸的是，操作系统微软 NuGet。服务器只能在 Windows 上运行，因为它需要。NET 框架。Nexus 是 Sonatype 的跨平台存储库解决方案。免费的操作系统运行在 Linux 和 Linux 容器中。SonaType 为在 OpenShift 上部署 Nexus 操作系统提供了基于 RHEL(和 CentOS)的模板。

我们创建一个`nexus`项目并导入模板:

```
$ oc new-project nexus
$ oc create -f https://raw.githubusercontent.com/sonatype/docker-rhel-nexus/master/OpenShift/nexus-rhel.json
```

创建对象后，初始构建开始。这可能需要很长时间。构建完成后，OpenShift 上就会有一个 Nexus 映像。现在，我们可以将这个映像部署到 OpenShift。该模板包括一个具有`readwriteonce`访问模式和指定大小(默认为 2Gi)的 PVC。如果您没有可用的持久存储，请按照[文档](https://docs.openshift.com/container-platform/3.6/install_config/persistent_storage/index.html)进行设置。

```
$ oc new-app --template=nexus -p APPLICATION_HOSTNAME=nexus.172.0.0.1.nip.io -p SIZE=2Gi
```

部署完成后，您可以使用浏览器访问具有指定主机名的 nexus 服务器。您可以使用默认密码`admin123`作为`admin`登录。

![nexus login](img/03e3b1e29bb1082fabc9d32cdbfc5ca5.png)

您将在 components 下找到两个 NuGet 服务器，`nuget.org-proxy`和`nuget-hosted`。

![nexus components](img/2d16e0e3a16663f0436a0e5115d2c383.png)

`nuget.org-proxy`是 NuGet.org 服务器[的缓存服务器。当代理服务器不包含请求的 NuGet 包时，它从 NuGet 服务器下载并存储它。当稍后请求存储的 NuGet 包时，代理服务器提供存储的包。使用`nuget.org-proxy`服务器不需要任何配置。让我们用 s2i-dotnetcore 这个服务器。](https://www.nuget.org/packages)

![copy repository url](img/491c14ac7aa00cbf49a84109a7916254.png)

## s2i。NET Core with nuget.org-proxy

![nuget api key](img/5764ded5f2f86f95bdab0dccdea83c6f.png)
[open shift s2i-dotnetcore builder](https://access.redhat.com/containers/#/registry.access.redhat.com/dotnet/dotnet-20-rhel7)有一个用于指定 NuGet 服务器的配置变量:`DOTNET_RESTORE_SOURCES`为构建环境变量。您可以在使用 OpenShift web 控制台创建应用程序时指定它:

![build environment](img/8c986b70fb7d0ce194f12d16d1e2646e.png)

或者您可以在使用“oc”命令时指定它，

```
$ oc new-app --template=dotnet-example --name=dotnet-demo --param=APPLICATION_DOMAIN="dotnet-demo.172.0.0.1.nip.io" --param=CONTEXT_DIR="/app" --param=NAME=dotnet-demo --param=SOURCE_REPOSITORY_URL=https://github.com/redhat-developer/s2i-dotnetcore-ex.git  --param=SOURCE_REPOSITORY_REF=dotnetcore-2.0 --param=DOTNET_STARTUP_PROJECT= --param=DOTNET_RESTORE_SOURCES=http://nexus.172.0.0.1.nip.io/repository/nuget.org-proxy/
```

构建现在将使用本地 NuGet 服务器。构建完成后，您可以在 nexus 控制台找到缓存的 NuGet 包。

![cached packages](img/b479401ff24202bba58f8a044ac12c5c.png)

## 添加您的私有 NuGet 包

当你想使用 nexus server 托管你的私有包时，你可以使用`nuget-hosted` server。您必须将您的私有包推送到这个 NuGet 服务器。为此，需要一个 NuGet APIKey。要找到您的 NuGet，请点击标题右侧的用户图标，然后选择`NuGet API Key`

![nuget api key](img/5764ded5f2f86f95bdab0dccdea83c6f.png)

您还必须在服务器管理员设置中将`NuGet API-Key Real`添加到领域中。以管理员用户身份登录后，点击标题中的齿轮图标，然后点击`Realms`并激活它。

![add nuget api key to realms](img/f9dd9d7b9059a430d82096e4e11cd282.png)

现在你可以把你的包推送到 NuGet 服务器了。如果你的包文件是`myawesomelibrary.nupkg`，命令如下。

```
$ dotnet nuget push -k  --source http://nexus.172.0.0.1.nip.io/repository/nuget-hosted/ myawesomelibrary.nupkg
```

Nexus 控制台向您显示您已经推送到服务器的包。

![pushed package](img/212918b610345b6f5289a4539e292af6.png)

## 结论

我们学习了在 OpenShift 上设置 SonaType Nexus OSS，以及如何将其用作 OpenShift s2i-dotnetcore builder 的本地 NuGet 服务器。如果您将 nexus 用于生产，您应该需要为 nuget 包提供足够的存储空间，并设置用户身份验证。

*Last updated: September 3, 2019*
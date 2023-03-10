# 怎么修。在 Red Hat OpenShift 上出现 NET Core 的“无法获得锁定文件访问”错误

> 原文：<https://developers.redhat.com/blog/2020/07/30/how-to-fix-net-cores-unable-to-obtain-lock-file-access-error-on-red-hat-openshift>

嗯，终于发生了。尽管增加了使用[容器](https://developers.redhat.com/topics/containers)和 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 的保证，但是旧的“它在我的机器上工作”的场景在我的[中出现了。NET Core](https://developers.redhat.com/products/dotnet/overview) (C#)代码。我创建的图像在我的本地电脑上运行良好——一台 Fedora 32 机器——但是当我试图在我的 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/getting-started) 集群中运行它时，它崩溃了。

错误是“无法在/tmp/NuGetScratch 上获得锁定文件访问权限。”让我们快速看一下发生了什么，然后我会解释我是如何修复它的。

## 身份问题

经过大量的网络搜索和与红帽的讨论。NET 核心工程师，我发现了底层问题。事实证明，在一个容器中，最初运行程序(使用`dotnet run`命令)所用的身份对于后续用户必须是相同的。

这个问题可能很容易理解，但解决办法是什么呢？

## 暂时的问题

解决方案不仅简单，而且是正确的做事方式。考虑我用来构建图像的初始 Dockerfile 文件:

```
FROM registry.access.redhat.com/ubi8/dotnet-31:3.1
USER 1001
RUN mkdir qotd-csharp
WORKDIR qotd-csharp
ADD . .

RUN dotnet publish -c Release

EXPOSE 10000
CMD ["dotnet", "run", "/bin/Release/netcoreapp3.1/publish/qotd-csharp.dll"]

```

请注意最后一行，这是运行映像时调用的命令:

```
CMD ["dotnet", "run", "./bin/Release/netcoreapp3.0/publish/qotd-csharp.dll"]

```

因为我用的是`dotnet run`。NET Core framework 正在尝试访问临时目录`/tmp/NuGetScratch`。因为构建映像的用户和试图运行它的用户不是同一个人，所以它在 Kubernetes 集群内部失败了。的。NET 运行时没有权限访问此目录。

## 更新 docker 文件(提示:不要运行)

解决方案很简单:我只使用了下面的 Dockerfile 文件。再次注意最后一行:

```
FROM registry.access.redhat.com/ubi8/dotnet-31:3.1
USER 1001
RUN mkdir qotd-csharp
WORKDIR qotd-csharp
ADD . .

RUN dotnet publish -c Release

EXPOSE 10000
CMD ["dotnet", "./bin/Release/netcoreapp3.0/publish/qotd-csharp.dll"]

```

更新后的文件不仅有效，而且效果更好。

因为库(`qotd-csharp.dll`)已经构建好了，当一个简单的`dotnet <path-to-dll>`是正确的时候，没有必要使用`dotnet run`命令。作为一个额外的好处，*它的启动速度是*的几百倍。

## 结论

即使在 Kubernetes 上使用容器化的应用程序,“它在我的机器上工作”的问题仍然会不时出现，尤其是在涉及权限的场景中。在这种情况下，不仅存在一种解决方法，而且这是运行映像的正确方法。把这归因于 PEBKAC 错误——即键盘和椅子之间存在的问题。我得到了教训。

关于此错误，您可以在 [Red Hat 知识库文章](https://access.redhat.com/solutions/5142731)中阅读更多信息。

*Last updated: August 6, 2020*
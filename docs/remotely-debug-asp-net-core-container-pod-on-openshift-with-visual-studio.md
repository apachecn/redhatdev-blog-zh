# 使用 Visual Studio 在 OpenShift 上远程调试 ASP.NET 核心容器 pod

> 原文：<https://developers.redhat.com/blog/2018/06/13/remotely-debug-asp-net-core-container-pod-on-openshift-with-visual-studio>

去年，我写了一篇博文[如何用 Visual Studio 代码](https://developers.redhat.com/blog/2017/10/23/remote-debug-asp-net-core-container-openshift-visual-studio-code/)在 OpenShift 上远程调试你的 ASP.NET 核心容器。今天，我将介绍如何使用 Visual Studio 从您的 Windows 计算机远程调试 pod。有时，您会遇到只在生产环境中发生的问题。远程调试 pod 使您能够调查这样的问题。

Visual Studio 和 Visual Studio 代码现在支持 SSH 作为远程调试的传输协议。如果远程主机接受 SSH 连接，Visual Studio 可以使用 Visual Studio 的默认功能进行远程调试。然而，您需要使用`oc`命令，而不是 SSH 客户端，例如 [putty](https://putty.org/) ，因为 Red Hat OpenShift pods 不允许通过 SSH 直接连接。通过 [MIEngine](https://github.com/Microsoft/MIEngine) 调试器，您可以使用任何命令进行 SSH 连接。

![](img/41c4201cf2870e46d9269193ab3fb125.png)

注意:
以下所有步骤已经在 Windows 10 和 OpenShift 3.9 上使用 Visual Studio 2017(15 . 7 . 2 和 15.8 preview2 版本)的组合进行了确认。

## 设置您的 ASP.NET 核心吊舱

设置 ASP.NET 核心 pod 的过程几乎与我之前为 Visual Studio 代码编写的过程相同。以下是总结。

### 调试版本

调试版本非常适合远程调试。如果您使用。NET Core s2i 映像，默认情况下，该版本是一个发布版本。您可以通过指定`DOTNET_CONFIGURATION` build 环境变量来切换到调试版本，如下所示:

```
$ oc set env bc DOTNET_CONFIGURATION=Debug

```

### 禁用运行状况检查

禁用健康检查不是强制性的，但建议禁用，因为在远程调试过程中，当进程暂停时，可能会重新启动 pod。您可以移除[准备就绪检查](https://docs.openshift.com/container-platform/3.9/dev_guide/application_health.html#container-health-checks-using-probes)以防止意外重启。

### 在 Pod 中安装 vsdbg 二进制文件

二进制文件是微软的[微软工程师的产品。您可以按如下方式在 pod 中安装`vsdbg`:](https://github.com/Microsoft/MIEngine)

```
$ oc rsh 
# curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v vs2017u5 -l ~/vsdbg

```

## 启动远程调试会话

详细步骤在 [MIEngine wiki](https://github.com/Microsoft/MIEngine/wiki/Offroad-Debugging-of-.NET-Core-on-Linux---OSX-from-Visual-Studio) 中有描述。我在这里描述一些要点。

首先，您需要创建一个启动配置文件。这类似于 Visual Studio 代码的`launch.json`文件，但并不完全相同。这里有一个例子。请替换`oc.exe`文件的路径和 pod 的名称，以便它们适合您的环境。

```
{
  "version": "0.2.0",
  "adapter": "c:\\path\\to\\oc.exe",
  "adapterArgs": "exec -i  -- /opt/app-root/vsdbg/vsdbg --interpreter=vscode",
  "languageMappings": {
    "C#": {
      "languageId": "3F5162F8-07C6-11D3-9053-00C04FA302A1",
      "extensions": [ "*" ]
    }
  },
  "exceptionCategoryMappings": {
    "CLR": "449EC4CC-30D2-4032-9256-EE18EB41B62B",
    "MDA": "6ECE07A9-0EDE-45C4-8296-818D8FC401D4"
  },
  "configurations": [
    {
      "name": ".NET Core Attach",
      "type": "coreclr",
      "cwd": "/opt/app-root/app",
      "processId": "1",
      "request": "attach",
      "justMyCode": false,
      "sourceFileMap": { "/opt/app-root/src/": "${workspaceRoot}" }
    }
  ]
}

```

然后，在 Visual Studio 中打开已经与正在运行的应用程序同步的项目。在 Visual Studio 中选择**视图- >其他窗口- >命令窗口**。您可以通过在命令窗口中执行命令`DebugAdapterHost.Launch /LaunchJson:""`来启动远程调试会话。

![](img/71ca60fb90c596094d0411f462add540.png)

您可以放入一个断点，这样进程将在该点暂停。

![](img/854a977d9c2529b09f713ca688f63dca.png)

现在您可以在源代码中看到变量的实际值。您最喜欢的 Visual Studio 调试功能在 OpenShift 上的 ASP.NET 核心中可用。您可以使用分步执行功能逐行执行代码。

*Last updated: November 22, 2022*
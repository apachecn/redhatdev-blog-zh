# 使用。Linux 容器中的 NET Core 2.0 和 RHEL

> 原文：<https://developers.redhat.com/blog/2017/09/05/using-net-core-2-0-and-rhel-in-linux-containers>

。NET Core 2.0 代表了。净核心开发工作。这是第三个版本(之前的版本是 1.0 和 1.1)，带来了近 20，000 多个 API 和更丰富、更深入的开发人员体验。用通俗的话说就是。网芯准备黄金时间。

这篇博文将向您展示使用的关键步骤和必要配置。NET Core 2.0 运行在 Linux 容器中的 RHEL 上。

## 挑战

从……开始。NET Core 2.0，微软正在使用元引用来获取 ASP.NET 程序所需的位。在 2.0 版本之前，您需要列出 ASP.NET 程序运行所需的每个库。虽然这个想法背后的理论听起来不错——你只需要你真正需要的部分——但在实践中，还是有一些问题。首先，你基本上想要 ASP.NET 的所有地方。这是一个宽泛的说法，但比没有更准确。

其次，也是更成问题的是，我们是否应该称之为“挑战”，以保持引用的最新和正确版本化。如果您在一个项目中引用 15 个库，那么根据定义，保持它们都更新要比只引用一个库多 15 倍的工作量。

请考虑以下情况。csproj 文件与第二个示例中生成的文件:

使用 ASP.NET 元包的 vs2ocp.csproj 文件:

`<Project Sdk="Microsoft.NET.Sdk.Web">`

`<PropertyGroup>`


`<PublishWithAspNetCoreTargetManifest>false</PublishWithAspNetCoreTargetManifest>`
`</PropertyGroup>`

`<ItemGroup>`
`<PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />`


`<ItemGroup>`
`<DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />`
`</ItemGroup>`
`<ItemGroup>`
`<None Update="standalone.sh" Condition="'$(RuntimeIdentifier)' == 'rhel.7-x64' and '$(SelfContained)' == 'false'" CopyToPublishDirectory="PreserveNewest" />`

`</Project>`

生成的代码:

`<Project Sdk="Microsoft.NET.Sdk.Web">`

`<PropertyGroup>`
`<TargetFramework Condition="'$()' == ''">netcoreapp2.0</TargetFramework>`
`<TargetFramework Condition="'$()' != ''"></TargetFramework>`
`<UserSecretsId Condition="'$(IndividualAuth)' == 'True' OR '$(OrganizationalAuth)' == 'True'">aspnet-vs2ocp-612A697F-5F13-4384-8F85-4B1214C705C6`
`<WebProject_DirectoryAccessLevelKey Condition="'$(OrganizationalAuth)' == 'True' AND '$(OrgReadAccess)' != 'True'">0`
`<WebProject_DirectoryAccessLevelKey Condition="'$(OrganizationalAuth)' == 'True' AND '$(OrgReadAccess)' == 'True'">1`
`</PropertyGroup>`
`<!--#if (IndividualLocalAuth && UseLocalDB) -->`

`<!--#endif -->`
`<ItemGroup Condition=" '$(IndividualLocalAuth)' == 'True' AND '$(UseLocalDB)' != 'True' ">`
`<None Update="app.db" CopyToOutputDirectory="PreserveNewest" />`

`<ItemGroup Condition="'$()' == ''">`
`<PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />`
`<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.0.0" PrivateAssets="All" Condition="'$(IndividualAuth)' == 'True'" />`
`<PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.0.0" PrivateAssets="All" Condition="'$(IndividualAuth)' == 'True'" />`
`</ItemGroup>`
`<ItemGroup Condition="'$()' != ''">`
`<PackageReference Include="Microsoft.AspNetCore" Version="2.0.0" />`
`<PackageReference Include="Microsoft.AspNetCore.Authentication.Cookies" Version="2.0.0" Condition="'$(IndividualAuth)' == 'True' OR '$(OrganizationalAuth)' == 'True'" />`
`<PackageReference Include="Microsoft.AspNetCore.Authentication.OpenIdConnect" Version="2.0.0" Condition="'$(OrganizationalAuth)' == 'True' OR '$(IndividualB2CAuth)' == 'True'" />`
`<PackageReference Include="Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore" Version="2.0.0" Condition="'$(IndividualLocalAuth)' == 'True'" />`
`<PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="2.0.0" Condition="'$(IndividualLocalAuth)' == 'True'" />`

`<PackageReference Include="Microsoft.AspNetCore.Mvc.Razor.ViewCompilation" Version="2.0.0" PrivateAssets="All" />`

`<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="2.0.0" PrivateAssets="All" Condition="'$(IndividualLocalAuth)' == 'True'" />`

`<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="2.0.0" Condition=" '$(IndividualLocalAuth)' == 'True' AND '$(UseLocalDB)' == 'True'" />`

`<PackageReference Include="Microsoft.VisualStudio.Web.BrowserLink" Version="2.0.0" Condition="'$(UseBrowserLink)' == 'True'" />`

`</ItemGroup>`

`<ItemGroup Condition="'$(NoTools)' != 'True'">`
`<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" Condition="'$(IndividualLocalAuth)' == 'True'" />`
`<DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.0" Condition="'$(IndividualAuth)' == 'True' OR '$(OrganizationalAuth)' == 'True'" />`
`<DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />`


`</Project>`

如您所见，元包做了很多繁重的工作。

为了使事情变得更快，这种库的组合被放入运行时存储中，一个预编译库的存储。这使得安装和启动更快，因为它们存储在本地。它们被认为是。机器上的 NET Core 安装。

## 挑战的核心

与。NET Core 2.0，是附带的构件之一。NET Core 是运行时包存储(它支持创建预编译的包缓存，减少部署大小和启动时间)。这个工件的目标是创建更小更快的[ASP.NET](http://asp.net/)应用程序的部署。然而，它引入了一个假设:任何 ASP.NET 的[应用程序都将假设运行时包存储存在。网芯。如果缺少运行时包存储，应用程序将在运行时失败。](http://asp.net/)

(更多关于包商店的背景细节，以及它是如何工作的，它能带来什么好处，可点击这里:[https://github.com/dotnet/designs/issues/8](https://github.com/dotnet/designs/issues/8))。

For Red Hat, this introduces a problem. We currently do not build [ASP.NET](http://asp.net/) from source. As a result, the [ASP.NET](http://asp.net/) Runtime Package Store is not present in our .NET Core. Any [ASP.NET](http://asp.net/) application - no matter if they are built using our SDK or Microsoft's SDK - will, therefore, fail to run on top of our distribution of .NET Core.

## 有解决的办法吗？

Yes. Red Hat and Microsoft are working very closely to streamline this situation. In the meantime...

## 开发者该怎么做？

No worries; of course we have a solution. And, to be honest, it's No Big Deal. It involves a small script and adding one tiny section to your project (e.g. *.csproj) file.For starters, add this section to your project file; in this example case, this is example.csproj:`<ItemGroup>`
`<None Update="standalone.sh" Condition="'$(RuntimeIdentifier)' == 'rhel.7-x64' and '$(SelfContained)' == 'false'" CopyToPublishDirectory="PreserveNewest" />`
`</ItemGroup>`Notice the reference to the script file "standalone.sh". This file must also be added to your project with the following contents:`#!/bin/bash``ASSEMBLY=example.dll``SCL=rh-dotnet20``DIR="$(dirname "$(readlink -f "$0")")"``scl enable $SCL -- dotnet "$DIR/$ASSEMBLY" "$@"`Note that you must change the `ASSEMBLY=` line with the name of your application.

## 三小步

That's it for the tweaks necessary. In every case, you'll simply:

1.  将微小部分添加到您的*。csproj 文件。
2.  将“standalone.sh”脚本添加到项目目录中。
3.  编辑脚本中一行。

## 构建代码

After that, you use `dotnet publish` to prepare your code for containers, using the following command:`dotnet publish -c Release -r rhel.7-x64 --self-contained=false`

## 建立形象

最后，添加一个包含以下内容的文件“docker file”——同样，调整文件名以适应您的应用程序:

`FROM registry.access.redhat.com/dotnet/dotnet-20-runtime-rhel7``ADD bin/Release/netcoreapp2.0/rhel.7-x64/publish/. /app/``WORKDIR /app/``EXPOSE 5000``CMD ["scl", "enable", "rh-dotnet20", "--", "dotnet",  "example.dll"]`

## 在容器中运行映像

Now we can build and run the application, such as this example:`docker build -t example .``docker run -d -p 5000:5000 --name example example`

## 结论

虽然有一些小的变化需要得到你的。NET Core 2.0 应用程序运行在基于 RHEL 的容器中，回报是值得的。随着云原生计算和微服务变得越来越普遍，这些知识将让你为这个新领域做出贡献。如果你想了解这一趋势的更多信息，请查看 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview/) ，这是一个用于运行可扩展微服务的平台即服务(PaaS)。

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: September 3, 2019*
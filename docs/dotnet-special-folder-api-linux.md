# 跨平台定位特殊文件夹。网络应用

> 原文：<https://developers.redhat.com/blog/2018/11/07/dotnet-special-folder-api-linux>

。NET 有用于定位特殊文件夹的 API，这些文件夹可用于应用程序和用户配置以及数据存储。它们提供了一种方便、可移植的方式，使跨平台的应用程序可以在不同的操作系统上找到合适的文件夹。我们将看看`Environment.GetFolderPath`、`Path.GetTempPath`和`Path.GetTempFileName`在 Linux 上的表现。

## 环境。GetFolderPath

`System.Environment`类有两个`GetFolderPath`重载:

```
public static string GetFolderPath (SpecialFolder folder);
public static string GetFolderPath (SpecialFolder folder, SpecialFolderOption option);

```

`SpecialFolder`是一个`enum`，其值为`ApplicationData`、`MyDocuments`和`ProgramFiles`。`SpecialFolderOption` `enum`有三个值:`None`、`Create`和`DoNotVerify`。这些控制文件夹不存在时的返回值。指定`None`会导致返回一个空字符串。指定`Create`会创建文件夹。并且`DoNotVerify`使得即使文件夹不存在也返回路径。

请注意，`SpecialFolder`和`SpecialFolderOption`嵌套在`Environment`类中，要直接使用它们，您应该在代码中添加一个`using static System.Environment;`语句。

为了使这个 API 跨平台工作。NET Core 需要将`SpecialFolder`值映射到一些位置。对于 Linux，这种映射基于[文件层次](https://www.freedesktop.org/software/systemd/man/file-hierarchy.html)和 [basedir-spec](https://specifications.freedesktop.org/basedir-spec/latest/) 。

| **特种部队** | **环境变量** | **配置文件** | **Linux 默认** |
| --- | --- | --- | --- |
| CommonApplicationData |  |  | /usr/share |
| 通用模板 |  |  | /usr/share/templates |
| 我的文档(*首页*) | 家 | 密码 | / |
| 用户概要 |  |  | *首页* |
| 应用数据 | XDG _ 配置 _ 主页 |  | *首页* /。配置 |
| LocalApplicationData | XDG _ 数据 _ 主页 |  | *首页* /。本地/共享 |
| 字体 |  |  | *首页* /。字体 |
| 桌面，桌面目录 | XDG 桌面目录 | user-dirs.dirs | *首页*/桌面 |
| 模板 | XDG _ 模板 _ 目录 | user-dirs.dirs | *首页*/模板 |
| 我的视频 | XDG _ 视频 _ 目录 | user-dirs.dirs | *首页*/视频 |
| 我们的音乐 | XDG 音乐目录 | user-dirs.dirs | *首页*/音乐 |
| 我的照片 | XDG _ 图片 _ 目录 | user-dirs.dirs | *首页*/图片 |

该表列出了中的所有映射值。网芯 2.1。其他值未被映射，并返回空字符串。返回值从左到右确定:首先检查环境变量，然后返回到配置文件，最后返回到默认值。

跨平台应用程序应该被限制使用映射值，或者当`GetFolderPath`返回一个空字符串时，它们应该能够退回到另一个位置。

从`HOME`环境变量中读取用户主文件夹。当该选项未设置时，将从系统用户数据库中读取主目录。可以有把握地假设，对于已知的用户。NET Core 将能够确定主目录。许多其他位置基于用户个人文件夹。有些可以使用环境变量来覆盖，有些可以通过在`*ApplicationData*/users-dirs.dirs`使用文件来覆盖。

在 Windows 上，默认情况下会存在大多数特殊文件夹。在 Linux 上可能不是这样。当文件夹不存在时，应用程序负责创建它。这可能需要对您的代码进行一些更改，以便使用带有`SpecialFolderOption`的重载。

例如，下面的代码确保在`LocalApplicationData`文件夹不存在时创建它。

```
// Use DoNotVerify in case LocalApplicationData doesn’t exist.
string appData = Path.Combine(Environment.GetFolderPath(SpecialFolder.LocalApplicationData, SpecialFolderOption.DoNotVerify), "myapp");
// Ensure the directory and all its parents exist.
Directory.CreateDirectory(appData);

```

## 路径。GetTempPath 和 Temp。GetTempFileName

`System.IO.Path`类有一个返回当前用户临时文件夹路径的方法:

```
public static string GetTempPath ();

```

Windows 应用程序可能认为此处返回的路径是用户特定的。这是因为实现选择了`USERPROFILE`环境变量。当变量未设置时，API 返回 Windows 临时文件夹。

在 Linux 上，实现返回`/tmp`。此文件夹已与其他用户共享。因此，应用程序应该使用唯一的名称，以避免与其他应用程序冲突。此外，由于该位置是共享的，其他用户将能够读取在此创建的文件，因此您不应将敏感数据存储在此文件夹中。第一个创建文件或目录的用户将拥有它。这可能会导致您的应用程序在尝试创建另一个用户已经拥有的文件或目录时失败。

`Temp.GetTempFileName`方法解决了创建文件的这些问题。它在`GetTempPath`下创建一个唯一的文件，该文件只对当前用户可读和可写。

在 Windows 上，`GetTempPath`返回的值可以使用`TMP` / `TEMP`环境变量来控制。在 Linux 上，这可以使用`TMPDIR`来完成。

在安装了`systemd`的系统上，比如 Fedora 和[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/download/)(RHEL)，有一个用户私有的临时目录，可以使用`XDG_RUNTIME_DIR`环境变量找到它。

## 结论

在本文中，您已经看到了在跨平台中使用`Environment.GetFolderPath`、`Temp.GetTempPath`和`Temp.GetTempFileName`的特性和局限性。NET 核心应用程序。

下面是一些附加的[。可能有帮助的网络核心文章](https://developers.redhat.com/blog/category/dot-net/):

*   [提高。NET Core Kestrel 使用 Linux 特定传输的性能](https://developers.redhat.com/blog/2018/07/24/improv-net-core-kestrel-performance-linux/)
*   [使用 OpenShift 进行部署。网络核心应用](https://developers.redhat.com/blog/2018/07/05/deploy-dotnet-core-apps-openshift/)
*   [在 Red Hat OpenShift 上运行微软 SQL Server](https://developers.redhat.com/blog/2018/09/25/sql-server-on-openshift/)
*   [固定。使用 HTTPS 的 OpenShift 上的 NET Core](https://developers.redhat.com/blog/2018/10/12/securing-net-core-on-openshift-using-https/)
*   [宣布。面向红帽平台的 NET Core 2.1](https://developers.redhat.com/blog/2018/06/14/announcing-net-core-2-1-for-red-hat-platforms/)

*Last updated: September 3, 2019*
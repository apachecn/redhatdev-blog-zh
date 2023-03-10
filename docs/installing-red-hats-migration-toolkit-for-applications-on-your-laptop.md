# 在您的笔记本电脑上安装 Red Hat 的应用程序迁移工具包

> 原文：<https://developers.redhat.com/blog/2020/12/15/installing-red-hats-migration-toolkit-for-applications-on-your-laptop>

如果你是一个考虑通过[容器化](https://developers.redhat.com/blog/2020/09/04/migrate-your-java-apps-to-containers-with-migration-toolkit-for-applications-5-0/)或者将它们迁移到一个更现代的应用服务器来使你的 Java 应用程序现代化的开发者，那么你可能知道 Red Hat 的[应用程序迁移工具包](https://developers.redhat.com/products/mta/overview)。本文通过将 migration toolkit for applications 直接安装在您的笔记本电脑上，帮助您开始使用它。有关该工具包的更多信息，请参见:

*   最近的[应用迁移工具包 5.1.0 功能更新](https://developers.redhat.com/blog/2020/12/08/spring-boot-to-quarkus-migrations-and-more-in-red-hats-migration-toolkit-for-applications-5-1-0/)。
*   我们的[在多个工作空间中分析单一 Java 应用的指南](https://developers.redhat.com/blog/2020/12/11/analyze-monolithic-java-applications-in-multiple-workspaces-with-red-hats-migration-toolkit-for-applications/)。

**注意** : Red Hat 的应用程序迁移工具包(前身为 Red Hat Application Migration Toolkit)是基于上游的开源[收尾项目](https://github.com/windup)。检查代码，看看它是如何工作的！

## 安装先决条件

要安装 migration toolkit for applications，您需要在笔记本电脑上安装 Java 开发工具包(JDK)或 Java 运行时环境(JRE)。我们在 Windows、macOS 和——当然还有——T2——Linux 上测试了这个工具。我们建议使用 [Red Hat 的 OpenJDK](https://developers.redhat.com/products/openjdk/download) 版本 8 或 11 来安装。

如果您碰巧使用的是 [Fedora](https://getfedora.org/) ，您可以使用以下命令安装 OpenJDK:

```
$ sudo dnf install java-11-openjdk.x86_64

```

输入以下内容，查看它是否正常运行:

```
$ java -version
openjdk version "11.0.8" 2020-07-14
OpenJDK Runtime Environment 18.9 (build 11.0.8+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.8+10, mixed mode, sharing)

```

这样，你就可以开始了。

## 下载应用程序迁移工具包

您可以在这里下载应用程序迁移工具包，在这里您会看到各种下载选项，如图 1 所示。出于演示目的，我们建议选择 **Web 控制台**选项。

[![The downloads page lists a variety of download options, including the web console download.](img/8a62af577fa88ae2e002d942bd2bbddd.png "img_5f6e19e353556")](/sites/default/files/blog/2020/09/img_5f6e19e353556.png)

Figure 1: Download options for the migration toolkit for applications.

**注意**:使用此[快速链接](https://developers.redhat.com/download-manager/file/migrationtoolkit-mta-web-distribution-5.0.1-with-authentication.zip)下载应用程序 5.0.1 web 控制台安装的迁移工具包。

## 解压缩工具包并运行它

如果您使用了提供的快速链接，那么您已经下载了一个压缩的 ZIP 文件。您可以使用 Fedora 文件管理器解压缩文件，如图 2 所示。

[![Right-click the ZIP file to uncompress it in Fedora Linux.](img/ac8708bbd6cc149499b91909d63012b9.png "img_5f6e1ba42b940")](/sites/default/files/blog/2020/09/img_5f6e1ba42b940.png)

Figure 2: Uncompress the downloaded ZIP file to a folder.

如果您愿意，可以使用 Fedora 命令行:

```
$ unzip migrationtoolkit-mta-web-distribution-5.0.1-with-authentication.zip

```

解压缩文件后，您可以通过输入刚刚创建的文件夹的名称`mta-web-distribution-5.0.1.Final`来运行它。对于 Linux 或 macOS，输入`run_mta.sh`。对于 Windows，输入`run_mta.bat`。下面是 Fedora 上的命令:

```
$ cd mta-web-distribution-5.0.1.Final/
$ ./run_mta.sh

```

## 浏览器中应用程序的开放迁移工具包

工具包运行后，将浏览器指向 http://localhost:8080。注意，我们已经用 Mozilla Firefox 和 Google Chrome 测试了这个软件。我更喜欢使用 Firefox，但是两种选择都可以。

您将自动登录，并将看到 migration toolkit for applications 欢迎页面，如图 3 所示。

[![](img/7c6d15097d7229db212fd2a695c1e265.png "img_5f6e1dd0090c0")](/sites/default/files/blog/2020/09/img_5f6e1dd0090c0.png)

Figure 3: The migration toolkit for applications welcome page.

看到欢迎页面意味着安装完成。现在，您可以直接在浏览器中使用该工具包。您可以应用 1，200 多条规则来分析 Java 二进制文件和 Java 代码，以获得大量可用的转换路径。您可能会从 Windup 项目存储库中测试各种示例 Java 应用程序二进制文件。

## 观看视频演示

我们制作了一个视频演示，帮助您开始使用迁移工具包 5.1 版。了解该工具包如何加速应用程序代码分析、支持工作量估算、加速代码迁移，以及帮助您将应用程序迁移到云和容器。

[https://www.youtube.com/embed/vt3jBGe2nLE?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/vt3jBGe2nLE?autoplay=0&start=0&rel=0)

## 结论

我们希望您喜欢使用应用程序迁移工具包来更新您的 Java 应用程序。此外，请查看 Red Hat 的虚拟化迁移工具包(将于 2020 年 12 月在技术预览中发布)和[的容器迁移工具包](https://www.redhat.com/en/events/webinar/migrating-effectively-with-migration-toolkit-for-containers)(以前的 Red Hat 容器应用程序迁移)。

*Last updated: December 14, 2020*
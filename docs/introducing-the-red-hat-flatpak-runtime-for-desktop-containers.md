# 桌面容器 Red Hat Flatpak 运行时简介

> 原文：<https://developers.redhat.com/blog/2020/08/12/introducing-the-red-hat-flatpak-runtime-for-desktop-containers>

多年来，希望为 Linux 开发桌面应用程序的应用程序开发人员不得不不仅为特定的 Linux 操作系统开发应用程序，还要为该操作系统的特定版本开发应用程序。无论是在服务器端还是在桌面上，开发人员都希望创建在开发和生产环境中可靠运行的应用程序。他们希望升级生产环境，而不必重新构建和重新验证每个正在运行的应用程序。

容器解决了服务器端应用程序的这些需求，但没有解决桌面应用程序的需求。这就是为什么我们需要 Flatpak，一个只用于桌面应用的容器系统。在本文中，我提供了 Flatpak 的概述，它与[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/products/rhel/overview)8.2 的集成，以及开发人员可以从新的 Red Hat Enterprise Linux Flatpak 运行时中得到什么。

**注意** : Red Hat 在 [Red Hat Enterprise Linux 8.2](https://www.redhat.com/en/blog/whats-new-red-hat-enterprise-linux-82) 中包含 Flatpak 运行时和软件开发工具包(SDK)镜像。集成 Flatpak 运行时将允许应用程序开发人员在 Red Hat Enterprise Linux 上构建容器化的桌面应用程序。

## Flatpak:一个专用的桌面容器系统

Flatpak 于 2018 年 8 月发布了第一个稳定版本，将服务器端容器中的相同思想和 Linux 内核技术应用于桌面。一旦一个桌面应用程序被打包成一个容器映像供 Flatpak 使用，我们简称之为 *Flatpak* ，它就可以可靠地跨不同的操作系统和版本使用。用户不必担心依赖关系的差异会导致应用程序行为不当或停止工作。

作为桌面容器的专用系统，Flatpak 实现了与桌面用户界面(UI)的透明和可靠的集成。从启动应用程序到允许应用程序访问窗口系统，再到访问用户的文件，大范围的集成是可行的。Flatpak 还引入了*门户*，它为应用程序提供了一种从其沙箱中访问主机服务的安全方式。门户使用户的决策成为用户交互的一部分。传统系统可能会在安装时询问应用程序是否允许打印，而对于门户，当显示打印对话框时，用户只需接受或取消即可。

## Flatpak 运行时

与服务器端容器一样，Flatpak 将应用程序与操作系统隔离开来。每个应用程序都使用自己的库，而不是操作系统的库。然而，为每个桌面应用程序拥有所有库的单独副本是不允许的，因此多个应用程序可以共享一个 Flatpak 运行时。运行时是一个包含系统级库和其他文件的文件系统映像。如图 1 所示，一个系统可以包含多种运行时，每个运行时由多个应用程序使用。

[![A diagram of a Linux operating system containing multiple Flatpak runtimes.](img/2321c9611387513a93158d3f7c98fce8.png "Operating system and Flaptaks")](/sites/default/files/blog/2020/04/OS-with-Flatpaks.png)

Figure 1\. A Linux operating system can contain multiple runtimes.

除了节省空间，运行时分离的另一大优势是处理安全更新。如果一个系统库出现了安全 bug，我们只需要更新运行时，不需要重新构建每一个应用。一个运行时不可能包含任何应用程序想要使用的所有库，所以库也可以和应用程序捆绑在一起。应用程序通常混合使用运行时库和捆绑库。

### 为什么我们需要 Red Hat 运行时

目前，许多现有的应用程序使用 Freedesktop 运行时，它提供了一组核心库。其他应用程序使用 GNOME 和 KDE 运行时，它们添加了特定于这些 Linux 环境的附加库。

Freedesktop 运行时得到了积极的维护——特别是， [Codethink](https://www.codethink.co.uk/) 付出了相当大的努力来保持它的最新。但是 Freedesktop 运行时的每个主要版本都只在有限的时间内得到支持。当达到这个极限时，您必须将您的应用程序移植到新版本。许多应用程序开发人员希望有一个更新时间更长的运行时。企业用户还想知道他们可以获得对主机系统和运行时组合的支持。

出于这些原因，Red Hat 的 Red Hat Enterprise Linux 团队已经决定发布 Red Hat Enterprise Linux (RHEL) 8.2，其中包含从 RHEL 软件包构建的 Flatpak 运行时。这个运行时遵循 [Red Hat Enterprise Linux 生命周期](https://access.redhat.com/support/policy/updates/errata)。我们打算从 RHEL 8 的 2019 年 5 月发布之日起继续提供 10 年的安全修复。

在不久的将来，我们将按照类似于[Red Hat Universal Base Images(UBI)](https://developers.redhat.com/products/rhel/ubi/)的条款发布运行时和 SDK。这些术语允许开发人员创建适用于许多 Linux 发行版的应用程序。用户不需要订阅 Red Hat Enterprise Linux。只要运行时在 Red Hat Enterprise Linux 主机上运行，就可以获得支持。

## 分发 Flatpaks 和运行时

Flatpak 并不要求所有的 flat pak 都来自单一的中心商店。相反，一个系统可以有任意数量的配置好的*遥控器*，作为平板包的来源。通过`flatpak`命令行工具(CLI)或 GNOME 软件浏览 Flatpaks 的用户将从所有已配置的遥控器上看到 Flatpaks。

目前，有两种类型的 Flatpak 遥控器。最初，Flatpaks 是从一个 [OStree](https://ostree.readthedocs.io/) 仓库分发的。OStree 是一个类似 Git 的系统，用于管理二进制操作系统树。它用于 Fedora CoreOS、Fedora Silverblue 和 Endless OS 等操作系统。您可以通过 HTTPS 导出一个 OStree 存储库用于远程下载。

最近，Flatpak 获得了从[容器注册中心](https://github.com/opencontainers/distribution-spec)下载 flat pak 作为[容器映像](https://github.com/opencontainers/image-spec)的支持。容器图像可以是开放容器倡议(OCI)容器图像，或者它们可以是传统的 Docker 图像。对这两种格式的支持允许使用广泛的容器注册中心。例如， [Red Hat Container Catalog](https://catalog.redhat.com/software/containers/search) 目前将 Flatpaks 作为 Docker 图像分发。

像 GNOME 软件这样的图形软件安装程序需要像描述和图标这样的信息。使用 Docker 和 OCI 注册协议，没有办法为所有图像收集这些所需的信息。因此，有必要为注册表上的 Flatpaks 提供一个单独的 JSON 索引。为 [Quay.io](https://quay.io/) 增加 Flatpak 索引支持的工作正在进行中。这项服务将为应用程序开发人员提供一种方便的方式来分发他们的工作。

## 安装 Red Hat Enterprise Linux Flatpak 运行时

下载 [RHEL Flatpak 储存库文件](https://flatpaks.redhat.io/rhel.flatpakrepo)以将储存库添加到您的系统中。在最近的 RHEL 或 Fedora 系统上，GNOME 软件应该提供将存储库添加到系统中。您也可以使用命令行来添加存储库:

```
$ flatpak remote-add rhel https://flatpaks.redhat.io/rhel.flatpakrepo

```

接下来，为您的 Red Hat Enterprise Linux 帐户提供凭证:

```
$ podman login registry.redhat.io
enter your username and password

```

**注意:**如果您没有 Red Hat 订阅，并且您想将运行时用于开发目的，那么您可以[注册一个免费的开发者订阅](https://developers.redhat.com/register/)。

波德曼只保存凭证，直到用户注销。如果要永久保存您的凭据，请输入:

```
$ cp $XDG_RUNTIME_DIR/containers/auth.json $HOME/.config/flatpak/oci-auth.json

```

如果您想为组织内的一组工作站启用 RHEL Flatpak remote，您应该使用一个[注册服务帐户](https://access.redhat.com/RegistryAuthentication#managing-registry-service-accounts-5)。您可以在`/etc/flatpak/oci-auth.json`在系统范围内为这样的帐户安装凭证。

现在安装 Flatpak 运行时:

```
$ flatpak install rhel com.redhat.Platform

```

RHEL Flatpak 遥控器将来也会包含应用程序。这些应用程序将出现在 GNOME 软件中，与来自其他 Flatpak 遥控器的应用程序一起浏览和安装。

## 创建应用程序映像

创建应用程序包括创建一个*应用程序映像*，在运行时，它将在`/app`挂载。应用程序映像与*运行时映像*一起，运行时映像安装在`/usr`。有各种方法来创建这些图像。例如，Fedora 用 Red Hat 包管理器(rpm)创建 Flatpaks。rpm 被重新构建并安装到 Fedora 的容器构建系统中的容器映像中。然而，创建 Flatpak 最典型的方法是使用 [flatpak-builder](https://docs.flatpak.org/en/latest/building-introduction.html) 工具。

### 使用 flatpak-builder

开发人员使用`flatpak-builder`工具和 SDK 映像来构建应用程序代码和任何所需的库。作为一名开发人员，您可以创建一个 JSON 或 YAML 清单，列出要构建和安装`flatpak-builder`的源代码。然后，`flatpak-builder`会在一个隔离的 Flatpak 运行时环境中构建源代码。不是在`/usr`安装一个正常的运行时，而是安装 SDK。SDK 是一个扩展的运行时，包含编译器和其他构建应用程序所必需的工具。每个运行时都有一个附带的 SDK。例如，要为`org.freedesktop.Platform`运行时构建一个应用程序，您可以使用`org.freedesktop.Sdk` SDK。要为`com.redhat.Platform`运行时构建一个应用程序，您可以使用`com.redhat.Sdk SDK`。

我们希望 RHEL Flatpak SDK 的范围类似于 Freedesktop SDK 的范围。这使得开发人员可以轻松地将为 Freedesktop 运行时构建的应用程序迁移到 RHEL 运行时。Freedesktop SDK 中几乎所有的构建工具和库在 RHEL Flatpak SDK 中也是可用的。

## 构建示例应用程序

作为一个简单的例子，我们将创建一个十六进制编辑器的 Flatpak。这个例子基于 github.com/flathub/org.gnome.GHex 的 T2，但是使用了 RHEL 的 Flatpak 运行时。

首先，创建一个名为`org.gnome.GHex`的目录。在该目录中，创建一个名为`org.gnome.GHex.yaml`的文件:

```
app-id: org.gnome.GHex
runtime: com.redhat.Platform
runtime-version: el8
sdk: com.redhat.Sdk
command: ghex
finish-args:
    # X11 + XShm
    - --share=ipc
    - --socket=x11
    # Wayland
    - --socket=wayland
    # Filesystem
    - --filesystem=host
modules:
  - name: ghex
    buildsystem: meson
    sources:
     - type: git
       url: https://gitlab.gnome.org/GNOME/ghex.git
       tag: 3.18.4
       commit: 228094e8d203ab62e49c33df2734fc9628c74775

```

安装 SDK 映像:

```
$ flatpak install rhel com.redhat.Sdk

```

构建应用程序:

```
$ flatpak-builder --repo=repo build-dir org.gnome.GHex.yaml

```

安装生成的应用程序:

```
$ flatpak --user remote-add --no-gpg-verify ghex-el8-repo repo
$ flatpak --user install ghex-el8-repo org.gnome.GHex

```

现在运行它:

```
$ flatpak run org.gnome.GHex

```

## 结论

Flatpak 允许应用程序开发人员以一种可以跨 Linux 操作系统和操作系统版本使用的方式打包他们的应用程序。通过使用包含在 Red Hat Enterprise Linux 8.2 中的 Flatpak 运行时和 SDK，开发人员可以在熟悉的平台上构建这样的应用程序，并利用 Red Hat Enterprise Linux 的生命周期。需要一个稳定和受支持的环境的用户可以从运行时与 Red Hat Enterprise Linux 主机的组合中获得这一点。

有关为应用程序创建构建清单的更多细节，请参见关于构建的 [Flatpak 文档。另外，请参见](https://docs.flatpak.org/en/latest/building.html)[用 Flatpak](https://opencontainers.org/posts/blog/2018-11-07-bringing-oci-images-to-the-desktop-with-flatpak/) 将 OCI 图像带到桌面，它描述了 Flatpak 如何与 OCI 图像和注册表一起工作。

*Last updated: August 17, 2020*
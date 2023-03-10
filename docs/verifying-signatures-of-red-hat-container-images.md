# 验证 Red Hat 容器图像的签名

> 原文：<https://developers.redhat.com/blog/2019/10/29/verifying-signatures-of-red-hat-container-images>

注重安全的组织习惯于使用数字签名来验证来自 Internet 的应用程序内容。一个常见的例子是 RPM 包签名。[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/rhel8/)默认情况下验证 RPM 包的签名。

在容器世界中，应该坚持类似的范例。事实上，Red Hat 的所有容器图像都经过了数字签名，而且已经有好几年了。许多用户没有意识到这一点，因为早期的容器工具并不支持数字签名。

在本文中，我将演示如何配置一个容器引擎来验证来自 Red Hat 注册中心的容器图像签名，以提高容器化应用程序的安全性。

在缺乏广泛接受的标准的情况下，Red Hat 设计了一种简单的方法来为其客户提供安全性。这种方法基于由标准 HTTP 服务器提供的*分离签名*。Linux 容器工具( [Podman](https://developers.redhat.com/blog/2019/04/25/podman-basics-cheat-sheet/) 、Skopeo 和 [Buildah](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/) )内置了对分离签名的支持，以及来自 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 的 CRI-O 容器引擎和 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)。

## 配置 Linux 容器工具以检查图像签名

将 Linux 容器工具配置为仅运行通过签名检查的容器映像是一个两步过程:

1.  在`/etc/containers/registries.d`下创建一个 YAML 文件，指定给定注册服务器的分离签名的位置。
2.  向`/etc/containers/policy.json`添加一个条目，指定验证给定注册服务器签名的公共 GPG 密钥。

Red Hat 将其容器图像的签名存储在 https://access . red Hat . com/web assets/docker/content/sigstore。Red Hat 用它用来签署 RPM 包的相同的 GPG 密钥签署它的图像。所有 Red Hat Enterprise Linux (RHEL)系统在`/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release`时都带有 Red Hat 的 RPM 公钥。

有了以上信息，您可以创建包含以下内容的`/etc/containers/registries.d/redhat.yaml`文件:

```
docker:
  registry.access.redhat.com:
    sigstore: https://access.redhat.com/webassets/docker/content/sigstore
```

并且，您可以编辑您的`/etc/containers/policy.json`,如下所示:

```
{
  "default": [
    {
      "type": "insecureAcceptAnything"
    }
  ],
  "transports":
    {
      "docker-daemon":
        {
          "": [{"type":"insecureAcceptAnything"}]
        },
      "docker":
        {
          "registry.access.redhat.com": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release"
            }
          ]
        }
    }
}
```

## 测试 Linux 容器工具拒绝签名检查失败的图像

验证设置的一个简单方法是使用[Red Hat Universal Base Image(UBI)](https://developers.redhat.com/blog/2019/10/09/what-is-red-hat-universal-base-image/)启动一个容器。如果以下命令失败，您可能在编辑容器运行时配置文件时出错:

```
$ sudo podman run --rm --name test registry.access.redhat.com/ubi8/ubi:8.0-199 date
Trying to pull registry.access.redhat.com/ubi8/ubi:8.0-199...Getting image source signatures
Checking if image destination supports signatures
Copying blob 567fcfc2ff35: 67.77 MiB / 67.77 MiB [=========================] 56s
Copying blob 188d0510bf14: 1.48 KiB / 1.48 KiB [===========================] 56s
Copying config a73bf97264a0: 4.43 KiB / 4.43 KiB [==========================] 0s
Writing manifest to image destination
Storing signatures
Sat Oct  5 17:24:31 UTC 2019
```

Podman 的输出看起来类似于您在没有配置签名检查的情况下得到的结果。确认签名是否被验证的一种方法是通过指定不正确的 GPG 密钥来强制验证错误。

如果在您的系统上配置了 EPEL 存储库，您可以如下编辑`/etc/containers/policy.json`以使用该存储库中的 GPG 密钥:

```
...
      "docker":
        {
          "registry.access.redhat.com": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8"
            }
          ]
        }
...
```

删除您已经下载的容器映像:

```
$ sudo podman rmi registry.access.redhat.com/ubi8/ubi:8.0-199
```

并尝试启动一个新容器。现在它失败了，因为容器图像签名与 GPG 键不匹配:

```
$ sudo podman run --rm --name test \
registry.access.redhat.com/ubi8/ubi:8.0-199 date
Trying to pull registry.access.redhat.com/ubi8/ubi:8.0-199...Failed
unable to pull registry.access.redhat.com/ubi8/ubi:8.0-199: unable to pull image: Source image rejected: None of the signatures were accepted, reasons: Invalid GPG signature: ...
```

错误信息实际上更长更可怕。我只展示了消息的前几行，但是很清楚为什么 Podman 不能提取图像。

在这个测试之后，记得修复你的`/etc/containers/policy.json`,让它为 Red Hat 容器映像和 RHEL RPM 包指定正确的 GPG 键。

## 用无根容器验证分离的图像签名

注意，我之前的命令使用`sudo`来运行`podman`。这就是你如何在 RHEL 和 CentOS 7.6 以及旧的 Fedora 版本上执行这些任务。非 RHEL 系统将不包括用于验证 RHEL 软件包的 GPG 公钥。这些密钥可以从[红帽客户门户](https://access.redhat.com/)【1】下载。

如果您使用的是最新的 RHEL、CentOS 或 Fedora 版本，您可能会支持无根容器[2]。这意味着你不再需要使用`sudo`。配置文件的更改保持不变。

## 将 Red Hat 容器镜像到私有注册表

许多组织不允许他们的服务器直接从 Internet 下载应用程序内容，不管该内容是否来自可信任的供应商以及是否经过数字签名。这些组织通常在内部部署一个私有注册服务器，并要求其服务器(有时是其开发人员)只从这个位置获取容器映像。

为了模拟这个场景，我将把 UBI 容器映像复制到我在 Quay.io 的个人帐户，并配置我的容器运行时来检查存储在那里的映像的映像签名。我会将我分离的签名存储到本地文件夹中，以避免设置 HTTP 服务器的需要。

当您在注册服务器之间复制容器映像时，您不能只复制它们分离的签名。这些签名依赖于完整的容器映像引用，其中包括注册表名称。因此，来自 Red Hat 的签名对于它们在私有注册表上的副本来说是无效的。

解决这个问题的一个方法是使用您的组织拥有的私有 GPG 密钥。然后，将公共 GPG 密钥复制到任何需要它的服务器和开发人员工作站。为了简单起见，我将使用我已经用来签署电子邮件的个人 GPG 密钥对。如果这是您第一次使用 GPG，请参阅参考资料[3]部分，获得一个关于生成 GPG 密钥对的很好的教程。

幸运的是，签名过程非常简单，它是由您用来将图像镜像到您的私有注册中心的相同的`skopeo copy`命令来处理的。

## 签名容器图像

用以下内容创建`/etc/containers/registries.d/quayio.yaml`文件，并用您在 Quay.io 的帐户替换“flozanorht ”:

```
docker:
  quay.io/flozanorht/ubi:
    sigstore: file:///var/lib/atomic/sigstore/
    sigstore-staging: file:///var/lib/atomic/sigstore/
```

你需要给你的用户写`/var/lib/atomic/sigstore/`的权限或者选择一个不同的文件夹。

然后，编辑您的`/etc/containers/policy.json`文件，如下所示:

```
...
      "docker":
        {
          "registry.access.redhat.com": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release"
            }
          ],
          "quay.io/flozanorht/ubi": [
            {
              "type": "signedBy",
              "keyType": "GPGKeys",
              "keyPath": "/etc/pki/rpm-gpg/flozano-pub"
            }
          ]
        }
...
```

为了将我的 GPG 公钥导出到`/etc/pki/rpm-gpg/flozano-pub`，我运行了以下命令:

```
$ gpg --export --armor flozano@redhat.com > /tmp/flozano-pub
$ sudo mv /tmp/flozano-pub /etc/pki/rpm-gpg/flozano-pub
```

最后，为了镜像 UBI 映像，`skopeo copy`命令需要`--remove-signatures`选项。如果没有它，Skopeo 会尝试将签名复制到目标注册服务器，而这仅在 OpenShift 内部注册中心上受支持。在拷贝操作过程中，仍然会验证源映像的签名。

接下来，登录您在 Quay.io 的帐户，使用`skopeo copy`命令创建 UBI 容器映像的镜像:

```
$ podman login -u flozanorht quay.io
Password:
Login Succeeded!
$ skopeo copy --remove-signatures --sign-by flozano@redhat.com \
docker://registry.access.redhat.com/ubi8/ubi:8.0-199 \
docker://quay.io/flozanorht/ubi:8.0-199
```

现在，作为验证容器运行时确实在检查容器映像签名的另一种方式，使用`podman`中的`--log-level debug`选项，使用 UBI 映像的副本启动容器:

```
$ sudo podman --log-level debug run --rm --name test \
quay.io/flozanorht/ubi:8.0-199 date
Trying to pull quay.io/flozanorht/ubi:8.0-199...DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration
DEBU[0000]  Using "docker" namespace quay.io/flozanorht/ubi
DEBU[0000]   Using file:///var/lib/atomic/sigstore/
...
DEBU[0001] GET https://quay.io/v2/flozanorht/ubi/manifests/8.0-199
DEBU[0002] IsRunningImageAllowed for image docker:quay.io/flozanorht/ubi:8.0-199
DEBU[0002]  Using transport "docker" specific policy section quay.io/flozanorht/ubi
DEBU[0002] Reading /var/lib/atomic/sigstore//flozanorht/ubi@sha256=c318fd9549dda67f3a1b3aa19b55b26b9dd42f597c3f2ee4181f0d9de16226de/signature-1
DEBU[0002] Reading /var/lib/atomic/sigstore//flozanorht/ubi@sha256=c318fd9549dda67f3a1b3aa19b55b26b9dd42f597c3f2ee4181f0d9de16226de/signature-2
DEBU[0002]  Requirement 0: allowed                     
DEBU[0002] Overall: allowed                            
Getting image source signatures
...
DEBU[0002] Started container 5148003a4eebc4ac256add74f127b4993217909189f6571b15fc77e883fbcd4f
Wed Oct  9 03:41:38 UTC 2019
DEBU[0002] Checking container 5148003a4eebc4ac256add74f127b4993217909189f6571b15fc77e883fbcd4f status…
DEBU[0002] Cleaning up container 5148003a4eebc4ac256add74f127b4993217909189f6571b15fc77e883fbcd4f
```

很难找到`podman`命令的输出，因为周围都是日志记录。调试消息可以更清楚地表明签名是有效的。但是，你可以看到它在声明`Requirement 0: allowed`之前读取了`quay.yaml`和`policy.json`配置文件，并从`/var/lib/atomic/sigstore/`读取了签名。

## 结论

在本文中，我展示了任何人都可以在没有复杂设置的情况下使用容器图像签名。进入生产场景，您需要做两件事:

*   将位于 registry.redhat.io 的基于 Red Hat 术语的注册表添加到您的配置中。
*   编写一个脚本，列出您要镜像的图像的所有标签，然后复制每个标签。Skopeo 足够聪明，不会复制你已经在你的私人注册表中的图像和图层。

RHEL7 用户可以使用`atomic`命令管理`/etc/containers`中与签名相关的文件。RHEL8 可以使用 podman 图像信任命令(不要与`podman images`命令混淆)做同样的事情。更多信息参见参考文献 4、5 和 6。

如果您不熟悉 Linux 容器工具(Podman、Buildah 和 Skopeo)，请参见下面的参考资料 7 和 8。

## 参考

1.红帽客户门户网站上的[产品签名密钥](https://access.redhat.com/security/team/key)

2.[在 RHEL 8.0 中使用无根容器技术预览版](https://www.redhat.com/en/blog/using-rootless-containers-tech-preview-rhel-80)

3.[生成新的 GPG 密钥](https://help.github.com/en/articles/generating-a-new-gpg-key)

4.[Red Hat 容器目录中的签名图像](https://www.redhat.com/en/blog/signed-images-red-hat-container-catalog)

5.[验证 Red Hat 容器注册表的图像签名](https://access.redhat.com/articles/3116561)

6.[https://github.com/containers/image/tree/master/docs](https://github.com/containers/image/tree/master/docs)

7.向 Buildah、Podman 和 Skopeo 问好

8.[针对 Docker 用户的 Podman 和 Buildah】](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)

*Last updated: January 5, 2022*
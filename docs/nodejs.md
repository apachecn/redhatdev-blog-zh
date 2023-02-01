# 在 Red Hat Enterprise Linux 上安装 Node.js

> 原文：<https://developers.redhat.com/hello-world/nodejs>

## 简介和先决条件

在本教程中，您将设置您的系统来安装来自 Red Hat Software Collections(RHS cl)的软件，它为 Red Hat Enterprise Linux 提供了最新的开发技术。然后，您将安装 Node.js v8 并运行一个简单的“Hello，World”应用程序。完成整个教程不到 10 分钟。

在您开始之前，您需要一个当前的 Red Hat Enterprise Linux 7(或 RHEL 6)服务器或工作站订阅，允许您从 Red Hat 下载软件并获取更新。开发者可以通过[developers.redhat.com](https://developers.redhat.com/)注册下载，获得免费的红帽企业版 Linux 开发者套件订阅用于开发目的。

如果您需要帮助，请参见[故障排除和常见问题解答](https://developers.redhat.com/products/softwarecollections/hello-world/#Troubleshooting)。

## 1.启用 Node.js Red Hat 软件集合

两分钟

在此步骤中，您将配置您的系统以从 RHSCL 软件存储库中获取软件，包括最新的动态语言。为命令行(CLI)和图形用户界面(GUI)都提供了说明。

Note for RHEL 6 users: If your system uses Red Hat Network (RHN) Classic instead of Red Hat Subscription Management (RHSM) for managing subscriptions and entitlements, please skip this step and follow the Installation chapter of the [Red Hat Software Collections Release Notes](https://access.redhat.com/documentation/en-us/red_hat_software_collections/). The instructions in this section are only for systems using RHSM. The remainder of this tutorial applies to systems running either RHSM or RHN Classic.

### 使用 Red Hat Subscription Manager GUI

使用*应用*菜单的*系统工具*组启动*红帽订阅管理器*。或者，您可以通过在命令提示符下键入`subscription-manager-gui`来启动它。

1.  从 subscription manager 的*系统*菜单中选择*存储库*。

2.  在存储库列表中，检查 *rhel-server-rhscl-7-rpms* 和 *rhel-7-server-optional-rpms(或 rhel-server-rhscl-6-rpms* 和*rhel-6-server-optional-rpms)*的*启用*列。如果您使用的是 Red Hat Enterprise Linux 的工作站版本，那么存储库将被命名为*rhel-workstation-RHS cl-7-rpms*和*rhel-7-workstation-optional-rpms(或 rhel-desktop-rhscl-6-rpms* 和*rhel-6-desktop-optional-rpms)*。注意:单击后，复选标记可能需要几秒钟才会出现在“已启用”列中。

如果您在列表中没有看到任何 RHSCL 存储库，则您的订阅可能不包括它。更多信息参见[故障排除和常见问题解答](https://developers.redhat.com/products/softwarecollections/hello-world/#Troubleshooting)。

### 从命令行使用 subscription-manager

作为根用户，您可以使用`subscription-manager`工具从命令行添加或删除软件仓库。使用`--list`选项查看可用的软件存储库，并验证您有权访问 RHSCL:

```
$ su -
# subscription-manager repos --list | egrep rhscl
```

如果您在列表中没有看到任何 RHSCL 存储库，则您的订阅可能不包括它。更多信息参见[故障排除和常见问题解答](https://developers.redhat.com/products/softwarecollections/hello-world/#Troubleshooting)。

如果您使用的是 Red Hat Enterprise Linux 的工作站版本，请在以下命令中将`-server-`更改为`-workstation-`:

```
# subscription-manager repos --enable rhel-server-rhscl-7-rpms
# subscription-manager repos --enable rhel-7-server-optional-rpms
```

或者对于 RHEL 6:

```
# subscription-manager repos --enable rhel-server-rhscl-6-rpms
# subscription-manager repos --enable rhel-6-server-optional-rpms
```

## 2.设置 Node.js 开发环境

两分钟

下一步，从 RHSCL 安装 Node.js v4:

```
$ su -
# yum install rh-nodejs8
```

注意:在 Node.js v8 中，JavaScript v8 运行时是内置的。不再需要单独的包装。

## 3.Hello World 和您的第一个 Node.js 应用程序

两分钟

在你的普通用户 ID 下，启动一个*终端*窗口。首先，使用`scl enable`将 Node.js v4 添加到您的环境中，然后运行 Node.js 来检查版本。

```
$ scl enable rh-nodejs8 bash
$ node --version
v8.6.0
```

下一步是创建一个可以从命令行运行的 Node.js 程序。使用文本编辑器如`vi`、`nano`或`gedit`创建一个名为`hello.js`的文件，内容如下:

hello.js

```
console.log("Hello, Red Hat Developer Program World from Node " + process.version)
```

保存它并退出编辑器。用`node`命令运行它:

```
$ node ./hello.js
Hello, Red Hat Developer Program World from Node v8.6.0
```

如果你得到错误:*节点命令没有找到*，你需要先运行`scl enable rh-nodejs8 bash`。

下一步是尝试一个稍微大一点的 Node.js 示例，它实现了一个微型 web 服务器。使用您喜欢的文本编辑器，创建一个名为`hello-http.js`的文件，包含以下内容:

你好-http.js

```
var http = require('http');
var port = 8000;
var laddr = '0.0.0.0';
http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello, Red Hat Developer Program World from ' +
	    process.version + '!\n');
    console.log('Processed request for '+ req.url);
}).listen(port, laddr);
console.log('Server running at http://' + laddr + ':' + port + '/');
```

保存它并退出编辑器。用`node`命令运行它:

```
$ node ./hello-http.js
```

现在使用 curl 或 Firefox 等浏览器连接 Node.js web 服务器`[http://localhost:8000/](http://localhost:8000/)`:

```
$ curl http://localhost:8000/
Hello, Red Hat Developer Program World from v8.6.0!
```

### 使用 RHSCL 包

RHSCL 中的软件包旨在允许同时安装多个版本的软件。为了实现这一点，需要使用`scl enable`命令将所需的包添加到您的运行时环境中。当`scl enable`运行时，它修改环境变量，然后运行指定的命令。环境变化只影响由`scl`运行的命令和从该命令运行的任何进程。本教程中的步骤运行命令`bash`来启动一个新的交互式 shell，以便在更新的环境中工作。这些变化不是永久性的。输入`exit`会用原来的环境回到原来的外壳。每次登录或开始新的终端会话时，都需要再次运行`scl enable`。

虽然可以更改系统配置文件，使 RHSCL 软件包成为系统全局环境的一部分，但不建议这样做。这样做可能会导致与其他应用程序的冲突和意外问题，因为路径中首先出现的是 RHSCL 版本，从而掩盖了包的系统版本。

#### 在您的开发环境中永久启用 RHSCL

要使一个或多个 RHSCL 包成为开发环境的永久组成部分，可以将其添加到特定用户 ID 的登录脚本中。这是推荐的开发方法，因为只有在您的用户 ID 下运行的进程才会受到影响。

使用您喜欢的文本编辑器，将下面一行添加到您的`~/.bashrc`:

~/.bashrc

```
# Add Node.js v8 from RHSCL to my login environment
source scl_source enable rh-nodejs8
```

进行更改后，您应该注销并重新登录。

当您交付使用 RHSCL 包的应用程序时，最佳实践是让您的启动脚本为您的应用程序处理`scl enable`步骤。您不应该要求您的用户更改他们的环境，因为这可能会与其他应用程序产生冲突。

### 接下来去哪里？

**使用 NodeSchool.io 教程学习 Node.js 和 JavaScript】**

现在你已经安装了 Node.js，使用来自 [nodeschool.io](http://nodeschool.io/#workshopper-list) 的教程来学习 Node.js 和 JavaScript。您需要已经运行了`scl enable rh-nodejs4 bash`或者已经将 Node.js 永久添加到您的开发环境中。

将 JavaScript 和 Node.js 教程安装到当前目录中:

```
$ npm install javascripting
$ npm install learnyounode
```

暂时将`node_modules/.bin`添加到您的路径中:

```
$ export PATH=$PATH:$PWD/node_modules/.bin
```

运行 JavaScript 教程:

```
$ javascripting
```

运行 Node.js 教程:

```
$ learnyounode
```

**查看 Nodejs.org 网站上的文档**
[http://nodejs.org/documentation/](http://nodejs.org/documentation)

**在 RHSCL 中找到附加的 Node.js 包**

```
$ yum list available rh-nodejs4-\*
```

**查看 RHSCL 中可用的完整产品包列表**

```
$ yum --disablerepo="*" --enablerepo="rhel-server-rhscl-7-rpms" list available
```

或者对于 RHEL 6:

```
$ yum --disablerepo="*" --enablerepo="rhel-server-rhscl-6-rpms" list available
```

## 想了解更多关于软件集合的信息吗？

*   阅读 [Red Hat 软件集合发行说明和打包指南](https://access.redhat.com/documentation/en/red-hat-software-collections/)

开发人员应该阅读打包指南，以更全面地了解软件集合如何工作，以及如何交付使用 RHSCL 的软件。成为红帽开发者:developers.redhat.com 红帽提供专家资源和生态系统，帮助您提高工作效率并构建出色的解决方案。在 developers.redhat.com 的[免费注册。](https://developers.redhat.com/register)

## 故障排除和常见问题

1.  **作为一名开发人员，我如何获得包含 Red Hat 软件集合的 Red Hat Enterprise Linux 订阅？**

    开发者可以通过[developers.redhat.com](https://developers.redhat.com/)注册下载，获得免费的红帽企业版 Linux 开发者套件订阅用于开发目的。我们建议您遵循我们的[入门指南](https://developers.redhat.com/products/rhel/get-started/)，该指南涵盖了使用您选择的 VirtualBox、VMware、Microsoft Hyper-V 或 Linux KVM/Libvirt 在物理系统或虚拟机(VM)上下载和安装 Red Hat Enterprise Linux。更多信息，请参见[常见问题:免费红帽企业 Linux 开发者套件](https://developers.redhat.com/articles/no-cost-rhel-faq/)。

2.  **我在我的系统**上找不到 RHSCL 存储库。

    一些 Red Hat Enterprise Linux 订阅不包括对 RHSCL 的访问。有关包含 RHSCL 的订阅列表，请参见[如何使用 Red Hat 软件集合(RHSCL)或 Red Hat 开发工具集(DTS)](https://access.redhat.com/solutions/472793) 。

    RHSCL 存储库的名称取决于您安装的是 Red Hat Enterprise Linux 的服务器版本还是工作站版本。您可以使用`subscription-manager`查看可用的软件存储库，并验证您是否有权访问 RHSCL:

    ```
    $ su -
    # subscription-manager repos --list | egrep rhscl
    ```

3.  RHSCL 软件包多久更新一次？

    RHSCL 中的产品包的支持生命周期是怎样的？

    一般来说，RHSCL 每年都会发布一个新版本。RHSCL 中的许多包都支持两到三年。参见[红帽软件系列产品生命周期](https://access.redhat.com/support/policy/updates/rhscl)。

4.  我可以在容器中使用 RHSCL 包吗？

    是的，许多 RHSCL 包可以作为 docker 格式的容器映像从 Red Hat Container Registry 获得。在[developers.redhat.com](https://developers.redhat.com/)上可以找到建造你的第一个容器的入门指南。

    用于构建 RHSCL 容器映像的`Dockerfiles`可用于构建定制容器映像。更多信息请参见[包装指南](https://access.redhat.com/documentation/en/red-hat-software-collections/)。

5.  我可以构建自己的使用 scl 机制来管理多个版本的包吗？

    是的。有关构建您自己的软件集合的信息，请参见 [RHSCL 打包指南](https://access.redhat.com/documentation/en/red-hat-software-collections/)。打包指南推荐给希望更全面了解软件集合如何工作的开发人员。

6.  有软件集合的开源社区吗？

    我如何贡献或参与软件集合？

    RHSCL 的上游开源社区可以在[softwarecollections.org](https://www.softwarecollections.org/about)找到，也称为 *SCLo* 。您可以与其他开发人员联系，在 SCLo 网站上为您的项目创建和托管新的集合。CentOS 项目下还有一个[软件集合特别兴趣小组](https://wiki.centos.org/SpecialInterestGroup/SCLo) (SIG)。

    注意:虽然 SCLo 可能是许多 RHSCL 包的源，但 Red Hat 只支持 RHSCL 中的包。

7.  **当我运行`yum install rh-nodejs8`时，由于缺少依赖**而失败。

    一些 RHSCL 集合需要可选 RPMs 存储库中的包，默认情况下不启用。有关如何启用可选的 RPMs 和 RHSCL 存储库，请参见上面的步骤 1。

8.  **如何才能知道安装了哪些 RHSCL 集合？**

    `scl --list`将显示已安装的收藏列表，无论是否启用。

    ```
    $ scl --list
    rh-nodejs8
    ```

9.  **如何发现 RHSCL 中是否有较新版本的 Node.js？**

    **如何了解当前 RHSCL 中 Node.js 的版本？**

    我启用了 RHSCL 存储库，但是我找不到本教程中列出的 Node.js 版本。

    使用以下命令查找具有匹配名称的包:

    ```
    # yum list available rh-nodejs\*
    ```

    注意:一些旧的集合不使用`rh-`前缀。要找到它们，可以省略前缀`rh-`或者用转义通配符`\*`来代替。

10.  **我已经从 RHSCL 安装了 rh-nodejs8，但是`node`不在我的路径中。**

    **找不到`node`命令。**

    RHSCL 不会改变系统路径。您需要使用`scl enable`来更改会话的环境:

    ```
    $ scl enable rh-nodejs8 bash
    ```

    更多信息请参见 [Red Hat 软件集合文档](https://access.redhat.com/documentation/en-US/Red_Hat_Software_Collections/2/)。

11.  **当我试图运行`node`时，我得到一个关于丢失共享库的错误。**

    这是因为没有先运行`scl enable`。`scl enable`运行时，除了设置命令搜索路径，还设置了共享库的搜索路径 LD_LIBRARY_PATH。

12.  如何卸载 RHSCL 软件包和任何依赖项？

    大多数集合都有一个`-runtime`元包，它导致主包和任何依赖项被安装。卸载`-runtime`包将导致不再需要的依赖包被删除。

    ```
    # yum uninstall rh-nodejs8-runtime
    ```

13.  **如何在“she bang”(#！)台词？**

    有了 scl-utils 包的当前版本，现在可以使用 Python、PHP、Node。来自“#”的 JS 和 Perl 解释器行，使用以下语法(或等效语法):

    ```
    #!/usr/bin/scl enable rh-python36 -- python
    ...

    ```

    (这个特性目前不适用于 Ruby 解释器。)

*Last updated: June 4, 2020*
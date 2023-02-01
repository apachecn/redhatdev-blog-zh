# 疑难解答和常见问题:Red Hat Enterprise Linux

> 原文：<https://developers.redhat.com/articles/troubleshooting-and-faq-red-hat-enterprise-linux>

## 下载和安装问题

1.  **作为一名开发者，如何获得免费的红帽企业版 Linux 订阅？**

    当您[通过【developers.redhat.com】的](https://developers.redhat.com/products/rhel/download/)[注册并下载](https://developers.redhat.com/)红帽企业版 Linux 服务器时，一个免费的红帽企业版 Linux 开发者套件订阅将自动添加到您的帐户中。更多信息，请参见[常见问题:免费红帽企业版 Linux 开发者套件订阅](https://developers.redhat.com/articles/no-cost-rhel-faq/)。

2.  我的系统无法从 Red Hat 下载软件或更新。

    您的系统必须使用`subscription-manager register`向 Red Hat 注册。你需要有一个最新的红帽订阅。注册您的系统会将其附加到您的订阅中。

3.  **我如何在安装后注册我的系统？**

    使用 Red Hat Subscription Manager，它可以作为图形工具从“系统”菜单启动，也可以从命令行启动:

    `# subscription-manager register --auto-attach`

    有关更多信息，请参见[如何使用 Red Hat Subscription Manager](https://access.redhat.com/solutions/253273) 向 Red Hat 客户门户网站注册和订阅系统。

4.  我有一个基于文本的登录屏幕，如何获得一个图形界面？

    在安装 Red Hat Enterprise Linux Server 的过程中，选择带有 GUI 软件选项的*服务器将安装一个完整的图形桌面，并将其配置为在引导时启动。向 Red Hat 注册您的系统后，您可以使用`yum install`安装图形桌面。以`root`用户的身份登录系统，然后使用以下命令:*

    ```
    # yum groupinstall 'Server with GUI'
    # yum install @gnome-desktop @x11 @internet-browser
    ```

    完成后，键入`systemctl reboot`重启系统。当系统重新启动时，您应该会看到一个图形化的登录屏幕。

5.  **我在安装过程中没有配置网络连接，如何在运行的系统上进行配置？**

    **注册失败，消息显示*subscription.rhn.redhat.com*不可达，我该如何解决？**

    如果您在安装过程中没有配置网络连接或配置不成功，请参见 [Red Hat Enterprise Linux 网络指南](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/index)了解使用图形或命令行工具配置网络的信息。

## 在 RHEL 发展

1.  如何安装 Red Hat Enterprise Linux 附带的 C/C++编译器？

    在安装过程中，选择*开发工具*软件选项将安装 C/C++编译器 GCC/G++和其他相关开发工具。向 Red Hat 注册您的系统后，您可以使用`yum install`安装这些工具。作为`root`用户登录系统，然后使用以下命令:

    ```
    # yum install @development
    ```

2.  **如何获得适用于 Red Hat Enterprise Linux 7 的较新 C/C++编译器？**

    **在哪里可以获得红帽企业版 Linux 7 上 C/C++开发的 IDE？**

    [红帽开发者工具集](https://developers.redhat.com/products/softwarecollections/overview/)提供最新、稳定、开源的 C 和 C++编译器以及包括 Eclipse 在内的补充开发工具。DTS 使开发人员能够一次编译应用程序，并跨多个版本的 Red Hat Enterprise Linux 进行部署。Red Hat 开发工具集使用 Red Hat 软件集合在`/opt/rh`中安装一组并行的包，它们不会覆盖 Red Hat Enterprise Linux 附带的系统包。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

    参见[在 RHEL 7 上使用最新的 GCC 和 Red Hat 开发工具集(DTS)](/products/developertoolset/hello-world/#rhel7)

3.  如何在 Red Hat Enterprise Linux 上获得 Perl、PHP、Python 和 Ruby 等语言的新版本？

    [Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview/)提供最新的稳定版本的动态语言、开源数据库和 web 开发工具，可以与 Red Hat Enterprise Linux 中包含的工具一起部署。Red Hat Software Collections 通过精选的 Red Hat Enterprise Linux 订阅提供，具有三年的生命周期，允许在不牺牲稳定性的情况下进行快速创新。

4.  **如何在红帽企业版 Linux 上获得 Python 3**？

    参见[在 RHEL 7 上使用 Python 3 和红帽软件集合](/products/softwarecollections/hello-world/#python)

5.  **如何在红帽企业版 Linux 上获取 node . js**？

    参见[在 RHEL 7 上使用 Node.js 与红帽软件集合](/products/softwarecollections/hello-world/#node)

6.  RHSCL 知识库不可用或在我的系统上找不到。

    存储库的名称取决于您是否安装了 Red Hat Enterprise Linux 的服务器、工作站或桌面版本。

    一些 Red Hat Enterprise Linux 订阅不包括对 RHSCL 的访问。有关哪些订阅包含 RHSCL 的信息，请参见[如何使用 Red Hat 软件集合(RHSCL)或 Red Hat 开发工具集(DTS)](https://access.redhat.com/solutions/472793) 。

    作为开发人员，您可以获得免费的 Red Hat Enterprise Linux 开发人员套件订阅，其中包括 RHSCL 和 DTS。更多信息请参见[常见问题:免费红帽企业版 Linux 开发者订阅](https://developers.redhat.com/articles/no-cost-rhel-faq/)。

7.  如何在 Red Hat Enterprise Linux 上安装 Eclipse？

    对于 C/C++开发，DTS 包括带有 C/C++开发工具(CDT)的 Eclipse，参见[使用 Eclipse](https://access.redhat.com/documentation/en-us/red_hat_developer_tools/2019.2/) 。

    对于 Java 开发， [Red Hat CodeReady Studio](https://developers.redhat.com/products/codeready-studio/overview/) 提供了一个单一的开发工具，为极高的生产力而定制，它构建在 Eclipse IDE 之上。开发者可以通过 developers.redhat.com[注册下载](https://developers.redhat.com/products/devstudio/download/)获得免费订阅。

*Last updated: January 9, 2023*
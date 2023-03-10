# hello World for Red Hat open shift Dev Spaces(以前的 CodeReady 工作区)

> 原文：<https://developers.redhat.com/crw-hw>

## 关于此页面

该页面有两个部分:

1.  安装 OpenShift 开发空间(开发空间)。
2.  在 OpenShift 开发空间中打开一个“Hello World”示例应用程序。

## 通过操作员中心安装

在 OpenShift 集群中创建 OpenShift Dev Spaces 实例有一个推荐的方法。浏览[管理指南](https://access.redhat.com/documentation/en-us/red_hat_openshift_dev_spaces/3.0/html-single/administration_guide/index#preparing-the-installation)了解更多信息。

## 通过命令行安装

**重要提示:OpenShift 4.10 或更高版本支持使用命令行创建 OpenShift Dev Spaces 实例。建议您使用位于操作中心的 OpenShift Dev Spaces 操作器来安装 Dev Spaces。**

### 1.下载并安装命令行工具，`dsc`

*3 分钟*

1.  导航到[https://developers . red hat . com/products/open shift-dev-spaces/download](/products/openshift-dev-spaces/download)。
2.  下载 3.0 版的 OpenShift Dev Spaces CLI 管理工具档案。
3.  提取档案。
4.  将提取的二进制文件放在您的`$PATH`中。

### 2.使用`dsc`在集群中创建一个 OpenShift Dev Spaces 实例

*10 分钟*

1.  使用集群管理员权限登录到您的 OpenShift 集群。
2.  从命令行运行命令:

    ```
    dsc server:start
    ```

这将在 OpenShift 名称空间**/project/open shift-Dev Spaces**中创建一个 OpenShift Dev Spaces 实例(如果它不存在，将被创建)。完成后，将返回 Dev Spaces URL。

## 运行一个示例工作区

登录到 Dev Spaces 实例后，选择任何示例项目。

## 为您自己的 Git 项目运行一个工作空间

登录到 Dev Spaces 实例后，输入 Git Repo URL 并点击 **Create & Open** 。如果您的项目还没有包含 devfile.yaml，将加载默认配置。要了解更多关于 devfiles 的信息，请参见 https://devfile.io/docs/devfile/2.1.0/user-guide/

*Last updated: August 9, 2022*
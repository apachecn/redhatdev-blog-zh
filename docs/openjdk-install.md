# 在 Windows 和 Linux 上安装 OpenJDK

> 原文：<https://developers.redhat.com/openjdk-install>

## Windows 安装

1.  在这里下载 OpenJDK [。](https://developers.redhat.com/products/openjdk/download/)
2.  要在 Red Hat Developer Studio 中使用 OpenJDK 11，请遵循 [Red Hat Developer Studio 说明](https://developers.redhat.com/products/devstudio/hello-world/)。

## Red Hat Enterprise Linux 安装

要在 Red Hat Enterprise Linux 上安装 OpenJDK 11:

1.  通过运行以下命令，确保您已启用可选通道:

    ```
    yum repolist all
    ```

    ```
    yum-config-manager --enable rhel-7-server-optional-rpms
    ```

2.  通过运行以下命令安装 OpenJDK 11 软件包:

    ```
    yum install java-11-openjdk-devel
    ```

*为了配置 Red Hat JBoss Developer Studio 或 Eclipse 以使用 OpenJDK 11，请遵循这些[指令](http://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Ftasks%2Ftasks-JREs.htm&cp=1_3_5)。

## Red Hat Enterprise Linux 6 安装

要在 Red Hat Enterprise Linux 6 上安装 OpenJDK 11:

1.  确保您订阅了基本频道。

2.  通过运行以下命令安装 OpenJDK 11 软件包:

    ```
    yum install java-11-openjdk-devel
    ```
# 在 Red Hat JBoss 企业应用平台 XP 2.0 上开发 Eclipse MicroProfile 应用程序

> 原文：<https://developers.redhat.com/blog/2021/01/12/develop-eclipse-microprofile-applications-on-red-hat-jboss-enterprise-application-platform-xp-2-0>

本文向您展示了如何安装支持 Eclipse MicroProfile 的[Red Hat JBoss Enterprise Application Platform(JBoss EAP)](https://developers.redhat.com/products/eap/overview)XP 2 . 0 . 0 GA。一旦您启用了 Eclipse MicroProfile，您将能够使用它的快速入门示例，通过 [Red Hat CodeReady Studio](https://developers.redhat.com/products/codeready-studio/overview) 开始开发您自己的 MicroProfile 应用程序。在这个演示中，您将学习两种构建和运行[micro profile Config](https://developers.redhat.com/cheat-sheets/microprofile-config)quick start 应用程序的方法。

## 正在安装 JBoss EAP XP 2.0.0 GA

要安装 JBoss EAP XP 2.0.0 GA:

1.  从[产品下载页面](https://developers.redhat.com/products/eap/download)下载以下软件:
    *   JBoss EAP XP 2.0.0 GA 管理器
    *   JBoss EAP 7.3.4 GA 补丁
    *   JBoss EAP XP 2.0.0 GA 修补程序
2.  应用 JBoss EAP 7.3.4 GA 补丁:

    ```
    $ patch apply /DOWNLOAD/PATH/jboss-eap-7.3.4-patch.zip

    ```

3.  设置 JBoss EAP XP 管理器:

    ```
    $ java -jar jboss-eap-xp-2.0.0.GA-manager.jar setup --jboss-home=/INSTALL_PATH/jboss-eap-7.3

    ```

4.  使用以下管理命令应用 JBoss EAP XP 2.0 补丁:

    ```
    $ java -jar jboss-eap-xp-2.0.0.GA-manager.jar patch-apply --jboss-home=/INSTALL_PATH/jboss-eap-7.3 --patch=/DOWNLOAD/PATH/jboss-eap-xp-2.0.0.GA-patch.zip

    ```

## 配置 CodeReady Studio

为了在 JBoss EAP 上启用 Eclipse MicroProfile 支持，我们首先需要为新安装的 JBoss EAP XP 2.0.0 实例注册一个运行时服务器。为此，我们将创建一个名为 Red Hat JBoss EAP 7.3 XP 2.0 的新 JBoss EAP 7.3 服务器。

服务器将使用新的 JBoss EAP 7.3 XP 2.0 运行时，它指向我们刚刚安装的运行时。JBoss EAP 7.3 XP 2.0 运行时使用`standalone-microprofile.xml`配置文件。

在**定义新服务器**对话框中选择或输入以下配置，如图 1 所示:

*   选择服务器类型**红帽 JBoss 企业应用平台 7.3** 。
*   将服务器的主机名设置为 **localhost** 。
*   输入**红帽 JBoss EAP 7.3 XP 2.0** 作为服务器名称。
*   点击**下一个**。

[![New Server dialog box with the specified options selected](img/bd618c30835b7dde476c40b44be3df37.png "crs-create_server_1")](/sites/default/files/blog/2021/01/crs-create_server_1.png)Figure 1: Define your new server.

Figure 1: Define your new server.

下一个屏幕邀请您创建一个新的服务器适配器。保持默认设置并选择**创建新的运行时**继续，如图 2 所示。

[![New Server dialog box for configuring the JBoss Runtime](img/6b05062587cba27bddbf97f51cc48bdb.png "New Server dialog box for configuring the JBoss Runtime")](/sites/default/files/blog/2021/01/crs-create_server_2.png)Figure 2: Configure your new server.

Figure 2: Select 'Create new runtime' to continue.

在下一个对话框中，您将配置新的运行时服务器。输入如图 3 所示的配置:

*   如果不想使用默认设置，请设置主目录。
*   确保您的执行环境设置为 **JavaSE-1.8** (但是您可以使用 **JavaSE-11** )。
*   如果不想要默认值，请更改服务器基本目录和配置文件的设置。
*   点击**完成**。

[![dialog box showing an overview of the server's settings](img/cc2b2bac282a8bc0ddd083be58f2715a.png "Figure 3: Set environment variables from the server Overview dialog box.")](/sites/default/files/blog/2021/01/crs-create_server_3.png)Figure 3: Set environment variables from the server Overview dialog box.

Figure 3: Configure the server runtime.

在运行 MicroProfile quickstarts 之前(参见图 5)，我们需要在运行时设置环境变量。导航至 Red Hat JBoss EAP 7.3 XP 2.0 服务器概述对话框，点击**打开启动配置**。您将看到设置环境变量的选项，如图 4 所示。

*   将**JAEGER _ REPORTER _ LOG _ SPANS**设置为 **true** 。
*   将**耶格 _ 采样器 _ 参数**设置为 **1** 。
*   将 **JAEGER_SAMPLER_TYPE** 设置为**常量**。

[![dialog box showing the newly created environment variables](img/9aa59820109e89742b109d520ac4c000.png "Figure 4: Configure your runtime’s environment variables.")](/sites/default/files/blog/2021/01/crs-create_server_4.png)Figure 4: Configure your runtime’s environment variables.

Figure 4: Configure your runtime’s environment variables.

## 运行 Eclipse MicroProfile 快速入门

Eclipse MicroProfile 快速入门提供了以下示例，您可以在安装的服务器上运行和测试这些示例:

*   Eclipse 微配置文件配置
*   Eclipse 微概要文件容错
*   Eclipse 微概要健康
*   Eclipse 微配置文件 JWT
*   Eclipse 微概要度量
*   Eclipse MicroProfile OpenAPI
*   Eclipse 微文件 OpenTracing
*   Eclipse MicroProfile REST 客户端

要打开快速入门，请打开项目浏览器，然后选择并导入`quickstart-parent` `pom.xml`。您将看到如图 5 所示的快速入门列表。

[![Project Explorer with quickstart-parent selected.](img/ea64e726015978c20ea4191daeee02bc.png "crs-projects_5")](/sites/default/files/blog/2021/01/crs-projects_5.png)Figure 5: Import quickstart-parent to turn on quickstarts.

Figure 5: Importing the quickstart parent turns on quickstarts.

## 运行 Eclipse MicroProfile 配置快速入门

有两种方法可以运行 Eclipse MicroProfile Config 应用程序:JBoss EAP XP Server 的裸机安装或可引导的 JAR 应用程序。我将向您展示如何做到这两点。

### 裸机安装

如果在裸机上安装`microprofile-config`,执行以下操作:

1.  右击**微配置文件配置**文件。
2.  选择**作为**运行。
3.  左键单击服务器上运行的 **1。**

如图 6 所示，这个配置启动部署了`microprofile-config`应用程序的服务器。

[![Runningthe microprofile-config project on local server](img/b539281f6b37c27514e2cd4d84eb93d3.png "crs-run_on_server_6")](/sites/default/files/blog/2021/01/crs-run_on_server_6.png)Figure 6: Run `microprofile-config` on bare metal installation.

Figure 6: A bare-metal installation.

### 将应用程序作为可引导 JAR 安装

现在，您可以将 JBoss EAP XP server 和 Eclipse MicroProfile 应用程序打包在一个可引导的 JAR 中，而不是在 JBoss EAP XP server 的裸机安装上运行您的应用程序。然后，您可以在 JBoss EAP XP 裸机平台上运行该应用程序。

出于演示的目的，我们为`microprofile-config`定义了一个 Maven 项目。该项目包括以下`bootable-jar`概况:

```
    ----
      <profile>
          <id>bootable-jar</id>
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.wildfly.plugins</groupId>
                      <artifactId>wildfly-jar-maven-plugin</artifactId>
                      <configuration>
                          <feature-pack-location>org.jboss.eap:wildfly-galleon-pack:${version.server.bootable-jar}</feature-pack-location>
                          <layers>
                              <layer>jaxrs-server</layer>
                              <layer>microprofile-platform</layer>
                          </layers>
                      </configuration>
                      <executions>
                          <execution>
                              <goals>
                                  <goal>package</goal>
                              </goals>
                          </execution>
                      </executions>
                  </plugin>
              </plugins>
          </build>
      </profile>
----

```

如您所见，我们选择了`jaxrs-server`和`microprofile-plateform`层来构建 JBoss EAP XP 服务器的精简版本，它封装了我们的应用程序。您所需要做的就是选择正确的概要文件，如图 7 所示:

1.  右键点击 **pom.xml** 。
2.  导航到 **Maven** 并选择**选择 Maven 配置文件**。
3.  检查**可启动 jar** 。

[![Dialog to select the apache Maven profile to use](img/3fea99f3db2d4992a28d283935399b98.png "crs-select_maven_profile_7")](/sites/default/files/blog/2021/01/crs-select_maven_profile_7.png)Figure 7: Select the Apache Maven Profile.

Figure 7: Select the Apache Maven profile.

接下来，我们需要使用 Apache Maven 构建可引导的 JAR。右击`pom.xml`。然后进入**运行方式**菜单，选择 **9 Maven 安装**。图 8 显示了这些选择。

[![Building the bootable jar using Apache Maven](img/a8491cb151f943e05ebe6cc5ec60f2a9.png "crs-build_bootable_jar_8")](/sites/default/files/blog/2021/01/crs-build_bootable_jar_8.png)Figure 8: Build the bootable jar.

Figure 8: Build the bootable JAR.

一旦构建了可引导的 JAR，我们需要从 CodeReady Studio 运行我们的应用程序。为此，我们将创建一个新的 Apache Maven 运行配置。如图 9 所示，右键单击`pom.xml`并导航到**运行方式**菜单，然后选择 **5 Maven 构建**。设置**org . wildfly . plugins:wildfly-jar-maven-plugin:Run**为目标，将执行重命名为**micro profile-config bootable Run**，然后点击 **Run** 。

[![Running te bootable jar using Apache Maven](img/696be528d1429c4b51b90b9fffbc018c.png "crs-run_bootable_jar_9")](/sites/default/files/blog/2021/01/crs-run_bootable_jar_9.png)Figure 9: Run the bootable jar.

Figure 9: Run the bootable JAR.

当服务器启动时，应用程序将作为根上下文在[http://localhost:8080/config/value](http://localhost:8080/config/value)处可用。

## 关闭服务器

关闭或终止终端不会停止正在运行的服务器，因此我们需要另一个运行配置来关闭服务器。最简单的方法是复制我们刚刚创建的运行配置，将其重命名为**micro profile-config bootable shut down**，并使用 goal**org . wildly . plugins:wildly-jar-maven-plugin:shut down**，如图 10 所示。

[![How to shutdown the server](img/3d5ef215cba949b0e6ba739bced333c7.png "crs-run_bootable_jar_10")](/sites/default/files/blog/2021/01/crs-run_bootable_jar_10.png)Figure 10: Shutdown the bootable jar.

Figure 10: Create a new run configuration to shut down the server.

## 结论

本文向您展示了如何安装带有 Eclipse MicroProfile 支持的 JBoss 企业应用程序平台 XP 2.0.0 GA。然后，我向您展示了使用 Red Hat CodeReady Studio 配置和运行 MicroProfile Config quickstart 项目的两种方法。

*Last updated: January 11, 2021*
# 在 Red Hat CodeReady Workspaces 2.5 中使用 IntelliJ Community Edition

> 原文：<https://developers.redhat.com/blog/2020/12/01/using-intellij-community-edition-in-red-hat-codeready-workspaces-2-5>

[Red Hat code ready work spaces](https://developers.redhat.com/products/codeready-workspaces/overview)(CRW)提供了一个默认的基于浏览器的 IDE，用于开发人员工作区。然而，该架构可以灵活地使用其他 ide，如 Jupyter Notebooks 和 Eclipse Dirigible。在本文中，您将学习如何使用社区版的 [IntelliJ IDEA](https://www.jetbrains.com/idea) 创建一个定制的工作区。

**注意**:您也可以应用本文中的指导创建一个免费的自助式 Eclipse Che 工作区，托管在 [che.openshift.io](https://che.openshift.io) 。

## 在 CodeReady 工作区中创建自定义工作区

我们将从在连接的 CodeReady 工作区环境中创建定制工作区的过程开始。请参见下一节，了解在空气间隙环境中设置自定义工作空间的说明。

### 步骤 1:添加自定义 IntelliJ IDEA 工作区

我们的第一步是从 devfile 安装 IntelliJ IDEA:

1.  打开 CodeReady 工作区，从仪表板的左上角选择**工作区**。
2.  点击**添加工作空间**。
3.  选择**自定义工作区**选项卡(如果尚未选择)。
4.  输入 CRW 2.5 IntelliJ devfile 的网址:
    [https://raw . githubusercontent . com/red hat-developer/code ready-work spaces/crw-2.5-rhel-8/dependencies/che-editor-IntelliJ-community/dev files/workspace . YAML](https://raw.githubusercontent.com/redhat-developer/codeready-workspaces/crw-2.5-rhel-8/dependencies/che-editor-intellij-community/devfiles/workspace.yaml)。
5.  点击**加载开发文件**。

图 1 显示了添加工作空间和加载 CRW 2.5 IntelliJ devfile 的对话框。

[![Screenshot of Create a Custom Workspace after loading a Devfile](img/47b391374a8b8aa46c3913917a23b4bf.png "CRW-Create-Workspace")](/sites/default/files/blog/2020/11/Screenshot-2020-11-20-at-11.23.39.png)

Figure 1: Add a workspace and load the IntelliJ devfile.

## 步骤 2:创建并打开工作区

向下滚动并点击**创建&打开**。加载工作空间后，将提示您接受 IntelliJ 社区版的许可条款。接受条款后，您将看到如图 2 所示的**欢迎使用 IntelliJ IDEA** 窗口。

[![Welcome screen for IntelliJ](img/6db97791f415ee28887e9bc06cbc78d1.png "Welcome-to-IntelliJ")](/sites/default/files/blog/2020/11/unnamed.png)Figure 2 - Welcome to IntelliJ IDEA

Figure 2 - Welcome to IntelliJ IDEA

在这里，单击**从版本控制**获取并输入 CodeReady Workspaces 项目 URL。我们示例的 URL 是 https://github.com/redhat-developer/codeready-workspaces 的。您可以使用 noVNC 剪贴板将 URL 从您的计算机粘贴到 IntelliJ 对话框中，如图 3 所示。

[![Shows the noVNC clipboard which allows you to copy and paste content such as a git repository URL](img/f2bd35c3938067f275cab14a3056ed68.png "CRW-noVNC")](/sites/default/files/blog/2020/11/pasted-image-0-1-1.png)Figure 3 - Using the noVNC clipboard

Figure 3: Using the noVNC clipboard to copy and paste the CodeReady Workspaces project URL.

接下来，点击**克隆**，等待项目加载。加载后，您可以打开一个项目文件并开始用 IntelliJ IDEA 编码。

### 其他提示

如果您不小心最小化了一个窗口，并且无法恢复，请参见[使用 Fluxbox 内容菜单](https://github.com/eclipse/che/issues/18318)启用工具栏的提示。启用工具栏后，您将能够再次找到该窗口。

如果 noVNC 界面使用的语言与您的初始设置不同，您可能需要[配置浏览器的语言设置](https://github.com/eclipse/che/issues/18319)。

## 在有空隙的环境中创建自定义工作空间

与连接环境相比，在空气间隙环境中创建自定义工作空间需要不同的方法和更多的步骤。

### 步骤 1:构建自定义插件注册表映像

首先，确保您已经将图像`quay.io/crw/plugin-intellij-rhel8:2.5`镜像到您的内部注册表。接下来，您将构建一个包含 IntelliJ:

1.  克隆具有 CRW 2.5 插件定义的 Git 存储库:

    ```
    git clone -b crw-2.5-rhel-8 https://github.com/redhat-developer/codeready-workspaces.git
    cd codeready-workspaces/dependencies/che-plugin-registry

    ```

2.  创建一个包含 IntelliJ 编辑器定义的新文件夹:

    ```
    mkdir -p v3/plugins/idea/intellij-community/2020.2.2/

    ```

3.  创建包含 IntelliJ 编辑器定义的`meta.yaml`文件:

    ```
    echo 'apiVersion: v2
    publisher: idea
    name: intellij-community
    version: 2020.2.2
    type: Che Editor
    displayName:  IntelliJ IDEA Community Edition
    title:  IntelliJ IDEA Community Edition (in browser using noVNC) as editor for Eclipse Che
    description:  IntelliJ IDEA Community Edition running on the Web with noVNC
    icon: https://resources.jetbrains.com/storage/products/intellij-idea/img/meta/intellij-idea_logo_300x300.png
    category: Editor
    repository: https://github.com/che-incubator/jetbrains-editor-images
    firstPublicationDate: "2020-09-25"
    spec:
      endpoints:
      - name: "intellij"
        public: true
        targetPort: 8080
        attributes:
          protocol: http
          type: ide
          discoverable: false
          path: /vnc.html?resize=remote&autoconnect=true&reconnect=true
    containers:
    - name: ideaic-novnc
      image: "quay.io/crw/plugin-intellij-rhel8:2.5"
      mountSources: true
      volumes:
      - mountPath: "/JetBrains/IdeaIC"
        name: idea-configuration
      ports:
      - exposedPort: 8080
      memoryLimit: "2048M"' > v3/plugins/idea/intellij-community/2020.2.2/meta.yaml
    ```

4.  指定最新版本为 2020 . 2 . 2:

    ```
    echo "2020.2.2" > v3/plugins/idea/intellij-community/latest.txt

    ```

5.  修补 docker file:

    ```
    echo '--- a/dependencies/che-plugin-registry/build/dockerfiles/Dockerfile
    +++ b/dependencies/che-plugin-registry/build/dockerfiles/Dockerfile
    @@ -12,7 +12,7 @@

    # Builder: check meta.yamls and create index.json
    FROM docker.io/alpine:3.11.5 AS builder
    -RUN apk add --no-cache py-pip jq bash wget skopeo && pip install yq jsonschema
    +RUN apk add --no-cache py3-pip jq bash wget skopeo && pip3 install yq jsonschema

    ARG LATEST_ONLY=false
    ARG USE_DIGESTS=false' | patch build/dockerfiles/Dockerfile

    ```

6.  构建并发布包含 IntelliJ 编辑器的新插件注册表映像:

    ```
    REGISTRY_URL=quay.io
    REGISTRY_ORG=crw
    REGISTRY_IMAGE_TAG=crw-2.5-rhel-8-intellij
    ./build.sh --organization ${REGISTRY_ORG} \
    --registry ${REGISTRY_URL} \
    --tag ${REGISTRY_IMAGE_TAG}
    docker push ${REGISTRY_URL}/${REGISTRY_ORG}/che-plugin-registry:${REGISTRY_IMAGE_TAG}

    ```

7.  更新 CRW 配置以使用插件注册表的新映像:

    ```
    CRW_NAMESPACE=workspaces
    oc patch checluster codeready-workspaces -n ${CRW_NAMESPACE} \
    --type=merge -p \
    '{"spec":{"server": {"pluginRegistryImage": "'${REGISTRY_URL}'/'${REGISTRY_ORG}'/che-plugin-registry:'${REGISTRY_IMAGE_TAG}'"}}}'

    ```

一旦对 CheCluster 定制资源(CR)打了补丁，插件注册表窗格将使用新的映像进行更新。

### 步骤 2:创建 IntelliJ 工作区

接下来，您将在 CodeReady 工作区中创建自定义工作区:

1.  打开 CodeReady 工作区，从 CRW 仪表板的左上角选择 **Get Started** 。
2.  选择**自定义工作区**选项卡(如果尚未选择)。
3.  将以下内容复制并粘贴到 devfile 文本区域:

    ```
    apiVersion: 1.0.0
    metadata:
    generateName: che-ideaic
    components:
    - type: cheEditor
      id: idea/intellij-community/2020.2.2

    ```

4.  点击**加载开发文件**。
5.  向下滚动并点击**创建&打开**。

## 结论

本文向您展示了如何在 CodeReady Workspaces 2.5 中使用 IntelliJ IDEA 创建自定义工作区。我们从连接环境的安装和设置开始，然后介绍了气隙安装。我们希望这些说明是有帮助的。如果您有任何问题或反馈，请在评论中告诉我们。

查看 Red Hat Developer 上的 [CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview) 产品页面，了解最新的新闻和教程。

## 承认

特别感谢 Nick Boldt 对本文的帮助。

*Last updated: May 18, 2021*
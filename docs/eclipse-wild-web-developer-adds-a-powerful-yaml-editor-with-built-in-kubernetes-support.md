# Eclipse Wild Web Developer 添加了一个具有内置 Kubernetes 支持的强大的 YAML 编辑器

> 原文：<https://developers.redhat.com/blog/2019/04/10/eclipse-wild-web-developer-adds-a-powerful-yaml-editor-with-built-in-kubernetes-support>

YAML 非标记语言(YAML)在过去几年里变得越来越流行。它是一种人类可读的基于文本的格式，用于指定配置信息，并在许多平台上使用，如 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://www.openshift.com/) 。

[Eclipse Wild Web Developer](https://marketplace.eclipse.org/content/eclipse-wild-web-developer-web-development-eclipse-ide) 是一个基于语言的扩展，为在 Eclipse IDE 中开发典型的 Web 和配置文件提供了丰富的开发体验。根据[项目页面](https://projects.eclipse.org/proposals/eclipse-wild-web-developer)，“Eclipse Wild Web Developer 依靠现有的主流和维护的组件来提供语言智能，超过了流行的配置文件如 TextMate 和协议如[语言服务器协议](https://microsoft.github.io/language-server-protocol/)或调试适配器协议。”

最近， [YAML 语言服务器](https://github.com/redhat-developer/yaml-language-server)已经集成到 Eclipse Wild Web Developer 中。这是一个功能丰富的 YAML 语言服务器实现，也支持包括 VSCode、Eclipse Che 和 Atom 在内的编辑器。这种集成将 Language Server 支持的所有特性(包括验证、自动完成、悬停支持和文档概述)都带到了 Eclipse 通用编辑器中，使得编写和维护 YAML 文件变得更加容易。

要获得这些特性，您可以在 Eclipse Marketplace 下载 [Wild Web Developer](https://marketplace.eclipse.org/content/eclipse-wild-web-developer-web-development-eclipse-ide) 。

### 演示

[https://www.youtube.com/embed/P9ETtuHiUco?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/P9ETtuHiUco?autoplay=0&start=0&rel=0)

### YAML 将军支持

如果您使用安装了 Wild Web Developer 的 Eclipse Generic Editor 打开任何`.yaml`或`.yml`文件，您会发现它具有您期望在基于语言的编辑器中找到的所有特性，包括:

*   语法突出显示
*   YAML 验证
*   文档大纲
*   格式化

### 内置 Kubernetes 和自定义模式支持

除了一般的 YAML 支持，Wild Web Developer 还提供内置的 Kubernetes 语法支持，以及通过 YAML 语言服务器对自定义模式的验证支持。

开箱即用，YAML 语言服务器将自动检查 SchemaStore 以提供基于匹配模式的语法、结构和值验证。您也可以通过转到*首选项- > YAML - > YAML 模式*并添加相应的模式来手动将模式关联到您的 YAML 文件。

在 *Schema* 部分，您可以指定内置的 kubernetes 定义，比如“Kubernetes”或“kedge”，或者任何指向有效模式的 URL。*全局模式*参数用于将模式匹配到文件路径。例如，您可以将其指定为/*以将模式应用于项目中的所有 YAML 文件，或者灵活地为不同的文件或目录指定不同的模式。

完成后，您就可以访问所有与模式相关的特性，例如:

*   自动完成:自动完成模式默认值的属性或节点值
*   悬停:将鼠标悬停在某个属性上会显示描述
*   模式验证:根据模式验证文件的结构和值

### 贡献的

如果你是一个使用 YAML 文件或任何典型网络语言的开发人员，那么看看 Wild Web Developer，看看它与你当前的工作流程相比如何。如果您愿意投稿，请随时提交任何问题、请求或评论，地址:[https://github.com/eclipse/wildwebdeveloper](https://github.com/eclipse/wildwebdeveloper)。

*Last updated: September 3, 2019*
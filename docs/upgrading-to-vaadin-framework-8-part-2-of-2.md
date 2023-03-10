# 升级到 Vaadin Framework 8(第 2 部分，共 2 部分)

> 原文：<https://developers.redhat.com/blog/2017/07/13/upgrading-to-vaadin-framework-8-part-2-of-2>

在本博客的[前一部分](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-1-of-2-)中，我谈到了让你的项目用最新的框架版本编译的最重要的步骤。

迁移已经通过这里提到的前三个步骤完成了，在这篇文章中，我将回顾迁移中最简单的步骤。步骤 4 和 5 涵盖了使用最新的 Framework 8 特性对项目进行现代化。如果您赶时间，也可以稍后再做，并且只对新的 Vaadin 代码使用新的 API。

1.  [升级 POM 文件中的依赖关系](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-1-of-2-#pom)
2.  [运行 Maven goal vaadin:upgrade8](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-1-of-2-#upgrade)
3.  [升级附加组件](https://vaadin.com/blog/-/blogs/upgrading-to-vaadin-framework-8-part-1-of-2-#addon)
4.  [升级非数据组件](#nondata)
5.  [升级数据组件](#data)
6.  [回到未来](#future)

和我之前的帖子一样，准备另一碗爆米花，召集所有的 Vaadin 专家，让我们看看迁移的第二部分:

[https://www.youtube.com/embed/zVT5zWLmwWE?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/zVT5zWLmwWE?autoplay=0&start=0&rel=0)

在这个视频中，策略发生了一点变化:我不再着急，我慢慢地升级项目的小部分，确保它总是能够编译并通过测试。您可以从升级小组件开始，然后转向更大的视图。您还可以很容易地找到不推荐使用的警告，从中您会发现大多数需要迁移的旧 Framework 7 组件。

## 4.升级非数据组件[ [01:17](https://www.youtube.com/watch?v=zVT5zWLmwWE&t=1m17s)

这简单明了。对于不使用数据绑定的组件，如布局和标签，大多数 API 没有变化。但是，您需要注意这些组件的某些新默认值。例如，默认情况下，VerticalLayout 和 HorizontalLayout 现在必须要有间距，以及最常用的边距设置。可能需要删除或添加一些代码，这取决于您对布局的预期。

## 5.升级数据组件[ [10:51](https://www.youtube.com/watch?v=zVT5zWLmwWE&t=10m51s)

 *我介绍了许多迁移数据组件的用例，让我们将它们分成三个部分:

#### 5.1 升级网格[ [11:10](https://www.youtube.com/watch?v=zVT5zWLmwWE&t=11m10s)

网格和所有数据组件现在都参数化了，它们不再处理容器，也不再有属性接口。在这个项目中，我必须转换 GeneratedPropertyContainer，以使用新的网格 API 来轻松生成列。我还需要为每一列手动指定一个标识符，以便以后在合并页脚列时能够引用它们。我发现一些与网格相关的组件有不同的包名，比如 FooterRow，但是它们的 API 完全相同。

#### 5.2 升级标签字段(来自 Viritin 插件)[ [20:06](https://www.youtube.com/watch?v=zVT5zWLmwWE&t=20m06s)

就像 Framework 8 中的核心字段组件一样，所有处理新数据绑定 API 的附加组件都发生了根本性的变化。在这个项目中，有一个旧的样式转换器直接应用于 LabelField。当迁移到 Framework 8 native API 时，转换器通过新的 Binder 类进行配置。

在整个过程中，我发现新的数据绑定 API 是编写转换器和验证器的更简单的方法。

该视频还展示了一个很好的例子，展示了如何使用新的绑定器将类的成员字段与给定的对象自动绑定。

#### 5.3 升级选项组[[30:12](https://www.youtube.com/watch?v=zVT5zWLmwWE&t=30m12s)]

现在有两个不同的组件，CheckBoxGroup 和 RadioButtonGroup，在这个项目中，我使用的是前者。CheckBoxGroup 用于多选，RadioButtonGroup 用于单选。不再需要 API 将多选设置为 true，但更重要的是，在生成标题或从组件中检索值时不再有不安全的强制转换。

## 6.回到未来

虽然视频中没有提到，但现在也可以安全地去掉兼容性库了。这样你的 war 大小会更小，浏览器需要的 JS 量也更小。对于最终用户来说，您的应用程序将部署得更快，初始加载也更快。要删除兼容性依赖关系，请将 POM 文件中的`vaadin-compatibility-server`依赖关系更改为`vaadin-server`。对于其他可能类似的`vaadin-compatibility-*`属地也是如此。

总的来说，在使用新的 Framework 8 组件时，我们可以看到许多增强，包括代码方面和性能方面。支持良好类型的`java.util.collections`而不是我们自己的容器-属性-项目接口使得代码更容易读写。同时，在内存和 CPU 方面，您将获得性能更好的服务器端。虽然不是必需的，但是升级所有其他组件以匹配最新的 API 和高级数据绑定技术是一个好的做法。

升级时有什么问题吗？

你有什么不顺心的事吗？

### 在下面的评论中分享你的经验吧！

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: July 12, 2017**
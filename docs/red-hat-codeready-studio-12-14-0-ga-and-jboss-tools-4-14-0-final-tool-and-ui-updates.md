# red Hat code ready Studio 12 . 14 . 0 . ga 和 JBoss Tools 4.14.0.Final:工具和 UI 更新

> 原文：<https://developers.redhat.com/blog/2020/04/20/red-hat-codeready-studio-12-14-0-ga-and-jboss-tools-4-14-0-final-tool-and-ui-updates>

在[上一篇文章](https://developers.redhat.com/blog/2020/04/20/red-hat-codeready-studio-12-14-0-ga-and-jboss-tools-4-14-0-final-openshift-application-explorer-view-feedback-loops-new-quarkus-tooling/)中，我介绍了 [JBoss Tools 4.14.0](https://tools.jboss.org/downloads/jbosstools/2019-12/4.14.0.Final.html) final 和[Red Hat code ready Studio 12.14](https://developers.redhat.com/products/codeready-studio/overview)for Eclipse 4.14(2019-12)，重点介绍了大的新特性:OpenShift 应用浏览器视图、反馈循环和新的 Quarkus 工具。本文主要关注许多较小的添加和更新。在这里，我将快速浏览一下 Hibernate Tools 和 Java Developer Tools (JDT)扩展中改进开发体验的新特性和小变化，这些特性和变化是针对 Java 13 更新的。我还将强调对平台视图、对话框和工具栏的 UI 更改。

## Hibernate 工具已更新

Hibernate 5.4 运行时提供程序现在包含 Hibernate 核心版本 5.4.12.Final 和 Hibernate 工具版本 5 . 4 . 1 . final。Hibernate 5.3 运行时提供程序现在包含 Hibernate 核心版本 5.3.15.Final 和 Hibernate 工具版本 5.3.15.Final

## Forge、JST 和 VPE 已弃用

Forge、Java Server Tools (JST)和可视化页面编辑器(VPE)在此版本中进行了更新，但我们没有计划在未来的版本中维护或添加新功能。我们可能会移除它们。我们还否决了 Red Hat JBoss Enterprise Application Server 4.3 和 5.0 的适配器。

## 视图、对话框和工具栏的平台变化

我们对视图、对话框和工具栏进行了大量的平台更改。

### 用于访问附加设置的新视图图标

几乎每个视图都有一个菜单，其中可能包含附加的配置设置，如过滤器、布局设置等。视图菜单经常被忽略，所以我们用一个更现代的等效物，一个垂直省略号，代替了旧的人字形图标。我们希望如图 1 所示的新图标能够帮助更多的用户找到并使用视图菜单。

[![A screenshot of the new vertical ellipsis icon.](img/84a589cf2906220261e6337df0350b97.png "img_5e6f32eec483c")](/sites/default/files/blog/2020/03/img_5e6f32eec483c.png)

Figure 1\. The new vertical ellipsis replaces the Chevron icon.

### “查找操作”用更新的用户界面取代了快速访问

以前被称为**快速访问**的功能已经被重命名为**查找动作**。我们认为新名称更好地描述了这个特性的功能。我们还改变了**查找操作**用户界面，以改善其使用和可访问性:

*   小部件项目现在是一个常规的工具栏项目(**按钮式的**)。
*   将显示一个图标。
*   右键单击工具项会起作用并显示典型的动作，如**隐藏**。
*   提案现在是一个以工作台为中心的常规对话框。

这些变化将极大地改善用户使用屏幕阅读器的体验，因为这些依赖于更加标准化的焦点状态。 **Find actions** 还利用了所有常见的对话框辅助功能，例如使屏幕和工具栏可移动和可调整大小。我们还改进了建议加载，以避免 UI 在加载过程中冻结。

让我们使用 **Find actions** 来运行快速文本搜索，并定位工作区文件。

#### 在“查找操作”中进行快速文本搜索

我们用快速文本搜索特性扩展了 **Find actions** ，它显示文件内容和提议中潜在的文本匹配。图 2 显示了正在进行的文本搜索。

[![A screenshot of a text search.](img/a758ceb6d919d9e7ff128104e4ac5c58.png "img_5e6f330979231")](/sites/default/files/blog/2020/03/img_5e6f330979231.png)

Figure 2\. A quick text search in progress.

在过去，如果你没有启动快速文本搜索包，你可能会错过这些匹配。现在，您可以使用**查找操作**来激活快速文本搜索。您需要做的就是找到并选择**Activate bundle for‘File content’**proposals 条目，如图 3 所示。

[![A screenshot of the selected &quot;Activate bundle for 'File content' proposals&quot; entry.](img/ef2f91608b93d0262333ab10f759984a.png "img_5e6f3321d1e98")](/sites/default/files/blog/2020/03/img_5e6f3321d1e98.png)

#### 在查找操作中定位工作区文件

**查找动作**现在可以从工作区列出匹配的文件名(类似于**打开资源**对话框)。选择后，文件在编辑器中打开，如图 4 所示。

[![A screenshot of the &quot;bignumber.js&quot; file selected in Find Actions.](img/af1232b5b73d6c8e7ad633ef04adc1c5.png "img_5e6f333e7cf85")](/sites/default/files/blog/2020/03/img_5e6f333e7cf85.png)

Figure 44\. Use Find actions to select a file and open it in the editor.

#### 简单的资源内联重命名

项目浏览器现在为简单的资源提供内联重命名。您可以使用 **F2** 快捷方式或**重命名上下文**菜单来重命名普通资源，只要其他文件不受重命名的影响。新的内联重命名特性如图 5 所示。

[![A screenshot of the inline-renaming feature selected in Project Explorer.](img/433fe6d4ca77ff6555ca5fd4669d590a.png "img_5e6f3391c8412")](/sites/default/files/blog/2020/03/img_5e6f3391c8412.png)

Figure 5\. The new inline renaming feature in Project Explorer.

### 文本编辑器

大多数文本编辑器现在可以显示问题标记，如错误、警告和信息标记。不用再四处寻找真正的错误信息。图 6 显示了一个例子。

[![Screenshot of A text editor identifies errors, warnings, and info markers highlighted in the code.](img/4d1e57c5f75eef094749b520298e0669.png "img_5e6f33b119949")](/sites/default/files/blog/2020/03/img_5e6f33b119949.png)

Figure 6\. A text editor identifies errors, warnings, and info markers in the code.

您还可以通过单击错误或警告消息来查看可用的快速修复，如图 7 所示。

[![Screenshot showing that clicking on an error or warning message pops up a quick fix.](img/8e8eb1e48e4fdbef71274f2a8b3c03d2.png "img_5e6f33cccf663")](/sites/default/files/blog/2020/03/img_5e6f33cccf663.png)

Figure 7\. Clicking on an error or warning message pops up a quick fix.

使用**通用- >编辑器- >文本编辑器**首选项页面启用该功能。您将被要求将**显示注释的代码挖掘**首选项设置为以下任一选项:

*   无(默认)
*   仅错误
*   错误和警告
*   错误、警告和信息

#### 在退格键上移除多个空格或删除

如果您在文本编辑器中使用**为制表符**插入空格选项，您现在可以更改退格键和 delete 键的行为，以便一次删除多个空格，就好像每个空格都是一个制表符一样。新的设置叫做**删除退格键上的多个空格/删除**。您将在文本编辑器首选项页面上找到它，如图 8 所示。

[![A screenshot of the &quot;Remove multiple spaces on backspace/delete&quot; option selected on the Text Editors preferences page.](img/64b153dba27e877d8b9650ed8fe60d8a.png "img_5e6f33e86ea5f")](/sites/default/files/blog/2020/03/img_5e6f33e86ea5f.png)

### 在调试视图中全部折叠

我们在**调试**视图中添加了一个新的**折叠所有**按钮。您可以使用这个新按钮来折叠所有的项目启动。图 9 显示了折叠前的启动工具栏。

[![A screenshot of the expanded launch toolbar.](img/678a2b2f2576dc25fe562d3406f9be29.png "img_5e6f340e73b15")](/sites/default/files/blog/2020/03/img_5e6f340e73b15.png)

Figure 9\. The launch toolbar before collapsing.

这是折叠后的列表，如图 10 所示。

[![A screenshot of the collapsed launch toolbar.](img/e166586b2d22d6d07d6efbf5ebec2905.png "img_5e6f3426edd2d")](/sites/default/files/blog/2020/03/img_5e6f3426edd2d.png)

Figure 10\. The launch toolbar after collapsing.

### 控制台视图中的控制字符

我们还更新了**控制台**视图中控制字符的解释。您现在可以设置控制台来解释反斜杠(`\b`)和回车(`\r`)控制字符。默认情况下，此功能是禁用的。您可以在**运行/调试- >控制台**首选项页面上启用它。图 11 展示了这个新特性的实际应用。

[![A screenshot of the console with code.](img/d26f5a7e2f6cadd78caa9169ba55cba1.png "img_5e6f3446be3df")](/sites/default/files/blog/2020/03/img_5e6f3446be3df.png)

Figure 11\. The console can be set to interpret control characters.

### 改进的主题和风格

我们改进了 UI 表单样式。ExpandableComposite 和 Section 中对级联样式表(CSS)的更新使您可以更好地控制这些元素的样式。在黑暗模式下，这两个元素现在可以更好地与其他表单元素集成。图 12 显示了这个版本之前的 UI 表单样式。

[![A screenshot of UI forms.](img/0bd155ebffff9df724c8daa415d6dd10.png "img_5e6f346b023d1")](/sites/default/files/blog/2020/03/img_5e6f346b023d1.png)

Figure 12\. The ExpandableComposite and Section forms prior to the new release.

图 13 显示了新版本中的这些表单。

[![A screenshot of UI forms.](img/35e796b6fac63b4d045ff80c13d6838c.png "img_5e6f3480b9534")](/sites/default/files/blog/2020/03/img_5e6f3480b9534.png)

Figure 13\. Updated UI forms in the new release.

我们还将**视角切换器**与普通的工具栏样式对齐，并移除了早期版本中的特殊功能。这一更改提高了工具栏外观的一致性。它还减少了该工具特定于操作系统(OS)的样式问题。图 14 显示了这个版本之前工具栏中的**透视图切换器**。

[![A screenshot of the old perspective switcher.](img/83fec9dfaec0dcc94bbe16452043a4e6.png "img_5e6f3498973da")](/sites/default/files/blog/2020/03/img_5e6f3498973da.png)

Figure 14\. Perspective switcher in the toolbar.

图 15 显示了新的外观。

[![A screenshot of the updated perspective switcher.](img/39f9066805b7aabb62eec5664947f457.png "img_5e6f34afbac9b")](/sites/default/files/blog/2020/03/img_5e6f34afbac9b.png)

Figure 15\. Perspective switcher updated for the new JBoss Tools 4.14.0 release.

我们还更新了深色主题，使用更一致的颜色和更少的灰色阴影。此外，小部件的样式不再基于选定的视图，这使得 UI 更加一致。

## 常规更新

我们已经对 JBoss Tools 4.14.0 进行了一些常规更新，我将快速回顾一下。

### 已更新至 Ant 1.10.7

Eclipse 采用了 Ant 版本 1.10.7，所以我们更新到了新版本。更新的结果是，`ant-ui-plugin`现在可以识别并验证 Ant `include`任务。从 Ant 1.8.0 开始，Ant 库中就有了`include`任务。

### 更新至 Java 13

Eclipse 4.14 版支持 Java 13。我们已经相应地更新到 Java 13 的 Java 开发工具(JDT)扩展。

### 预览 JDK 13 中的两个新语言功能

新版本包含的两个显著特性是 [JEP 354:切换表达式(预览)](https://openjdk.java.net/jeps/354)和 [JEP 355:文本块(预览)](https://openjdk.java.net/jeps/355)。注意，这些是预览语言功能，所以如果你想使用它们，你需要打开**启用预览**选项。参见 [Java 13 Examples wiki](https://wiki.eclipse.org/Java13/Examples) 获取对该支持的非正式介绍。

我们添加了一个键盘快捷键( **Ctrl+Shift+'** )，用于在 Java 编辑器中创建文本块。注意，新的键盘快捷键只适用于兼容 Java 13 或更高版本的 Java 项目，并且选择了 **Enable preview** 选项。此外，只有当编辑器中的选择不是字符串、注释或文本块的一部分时，此功能才有效。考虑图 16 中的例子。

[![Screenshot of a preview.](img/eb5873942d455051ce0e7262a994c567.png "img_5e6f34e931a8f")](/sites/default/files/blog/2020/03/img_5e6f34e931a8f.png)

当您按下键盘快捷键时，您将看到如图 17 所示的内容。

[![Screenshot of another preview.](img/62c793b5bc27b1effb78f350fd944400.png "img_5e6f34ffbcf1d")](/sites/default/files/blog/2020/03/img_5e6f34ffbcf1d.png)

Figure 17\. A preview for text blocks.

您还可以在文本块中包含选定的文本，如图 18 所示。

[![Screenshot of a selected code block.](img/3420e620aa90786e4ec6c7680435b926.png "img_5e6f351ca2a92")](/sites/default/files/blog/2020/03/img_5e6f351ca2a92.png)

Figure 18\. Selected code in a text block.

图 19 显示了执行快捷方式时您将看到的内容。

[![Screenshot of new code after the shortcut has been applied.](img/9009d203863b7a012824445582a74cbc.png "img_5e6f353e5253e")](/sites/default/files/blog/2020/03/img_5e6f353e5253e.png)

Figure 19\. Preview text in a text block.

### Java 编辑器的改进

我们还改进了 Java 编辑器本身，从删除不必要的数组创建的新清理动作开始。这个新动作删除了对`varargs`参数的显式数组创建，如图 20 所示。

[![Screenshot of the Java Editor selected to remove unnecessary arrays.](img/48e28b3ce373211431397d2157864db5.png "img_5e6f3560dfd3a")](/sites/default/files/blog/2020/03/img_5e6f3560dfd3a.png)

Figure 20\. The new cleanup action in the Java Editor.

考虑图 21 所示的代码。

[![A screenshot of code with unnecessary arrays.](img/ff8eb49e05530d0f648eaefe42b9fd76.png "img_5e6f3577c6202")](/sites/default/files/blog/2020/03/img_5e6f3577c6202.png)

Figure 21\. Messy code in the Java Editor.

这里是清理后的代码，如图 22 所示。

[![A screenshot of the same code with arrays removed.](img/3bbc266579ffda8422fdb694edcfd2f1.png "img_5e6f35911b1b9")](/sites/default/files/blog/2020/03/img_5e6f35911b1b9.png)

Figure 22\. Cleaned-up code in the Java Editor.

#### 减少双重否定

Java 编辑器有一个新的清理动作叫做**下推否定**，它通过还原算术表达式来减少双重否定，如图 23 所示。例如，`!!isValid;`变成了`isValid;`,`!(a != b);`变成了`(a == b);`。

[![A screenshot of the selection for push-down negation.](img/c714b787dfff0cb38f6c289f288b0dc4.png "img_5e6f35ab50f8e")](/sites/default/files/blog/2020/03/img_5e6f35ab50f8e.png)

#### 空 Java 源文件的新模板

我们已经制作了一些基本模板来填充空的 Java 源文件。您将使用图 24 所示的**内容助手**弹出窗口找到新模板。

[![Screenshot of a quick assist for creating classes.](img/1c2e8d48425961278382314d884a82f6.png "img_5e6f35c91365a")](/sites/default/files/blog/2020/03/img_5e6f35c91365a.png)

Figure 24\. Create a new class in an empty Java source file.

#### 后缀完成

后缀完成是一项新功能，它允许将某些类型的语言结构应用于以前输入的文本。例如，输入`"input text".var`并选择 **var -创建一个新的变量**建议，结果是`String name = "input text"`。图 25 显示了一个不同的选项。

[![Screenshot of a selection for &quot;for - Creates a for statement.&quot;](img/2265977acf5a3343c4f615833782388c.png "img_5e6f35e6138c5")](/sites/default/files/blog/2020/03/img_5e6f35e6138c5.png)

#### 一个新的快速修复资源尝试

我们添加了一个 quickfix 来为选定的代码行创建一个`try-with-resources`语句。选定的行必须以一个或多个实现`AutoCloseable`的对象的声明开始。这些声明将作为*资源*添加到`try-with-resources`语句中。如果您选择了一个或多个不包含合格资源的语句(例如没有实现`AutoCloseable`的`Object`，那么所有选择的语句都将被放入`try-with-resources`主体中。

例如，在应用`try-with-resources`之前考虑以下方法，如图 26 所示。

[![Screenshot of a method before applying try-with-resources.](img/e2c1739a8f02c9d497b0217f2fa3716f.png "img_5e6f360367016")](/sites/default/files/blog/2020/03/img_5e6f360367016.png)

Figure 26\. A method before applying try-with-resources.

选择方法中的所有行，然后选择键盘快捷键 **Ctrl+1** 。从图 27 所示的下拉列表中点击**用 try-with-resources** 包围。

[![A screenshot of the selection &quot;Surround with try-with-resources&quot;.](img/eab83f655da637f13ae49910ec578d28.png "img_5e6f362aecdf2")](/sites/default/files/blog/2020/03/img_5e6f362aecdf2.png)

代码将被更新，如图 28 所示。

[![A screenshot of the method surrounded by try-with-resouces.](img/49d331267ecbdd5512d3e0525b3287d6.png "img_5e6f3644bdf4f")](/sites/default/files/blog/2020/03/img_5e6f3644bdf4f.png)

Figure 28\. The method is now surrounded by try-with-resources.

#### Javadoc:对数据模块进行标记检查

我们增加了对检查`module-info.java`文件的 Javadoc 的支持，并根据编译器的设置报告丢失和重复的`@uses`和`@provides`标签。在**首选项- > Java- >编译器- > Javadoc** 访问该特性。图 29 显示了识别丢失的`@uses`标签的选项。

[![A screenshot showing an alert for missing @uses tag.](img/e7ddb7e492ae1e4a5c438c2578a845bb.png "img_5e6f36a1e2113")](/sites/default/files/blog/2020/03/img_5e6f36a1e2113.png)

Figure 29\. Identity and fix a missing @uses tag.

### Java 格式化程序的更新

我们还对 Java 格式化程序进行了更新，接下来我将对此进行描述。

#### 文本块格式

代码格式化程序现在可以处理 Java 13 中添加的新的文本块(预览)特性。它由**文本块缩进**设置控制，可在配置文件编辑器的**缩进**部分找到。路径是:**首选项- > Java- >代码样式- >格式化程序- >编辑…** 。

默认情况下，文本块行的缩进方式与换行代码行相同；也就是说，相对于起始缩进(或者您在**换行**部分中为换行设置的默认缩进)有两个额外的制表符。您还可以设置您的首选项，只使用一个制表符进行缩进(**缩进一个**)，将所有行对齐到左引号的位置(**缩进到第**列)，或者保留原始格式(**不要触摸**)。

[![Screenshot of a selection to &quot;Default for wrapped lines&quot;.](img/82a7c24167da5b9296a192ee0ea2903a.png "img_5e6f36bf4069b")](/sites/default/files/blog/2020/03/img_5e6f36bf4069b.png)

Figure 30\. Select a default indentation for wrapped lines.

图 30 显示了默认换行的选项。

#### Javadoc 标记之间的空白行

代码格式化程序现在可以按类型将 Javadoc 标签分成组(例如，`@param`、`@throws`和`@returns`)，并用空白行分隔这些组。你可以在**注释> Javadocs** 部分打开这个特性，通过勾选**不同类型标签之间的空白行**框。

### not 运算符后的空格

一个新的设置控制是否在*而不是* ( `!`)运算符后添加一个空格，独立于其他一元运算符。要找到它，展开**空白- >表达式- >一元**操作符并转到最后一个复选框，如图 31 所示。

[![Screenshot of the selection as a space after the &quot;not&quot; (!) operator.](img/07bd6864cde8c8e16af0ea29602ae543.png "img_5e6f36de06b97")](/sites/default/files/blog/2020/03/img_5e6f36de06b97.png)

Figure 31\. Selection to add a space after the not (!) operator.

### JUnit 执行环境的更新

我们已经为`org.eclipse.jdt.junit.runtime` J2SE-1.5 包更新了包所需的执行环境(BREE)。

### 在 JDT 调试器中处理异常断点

我们为异常断点添加了一个新的工作区首选项:**循环异常实例的挂起策略**控制同一个异常实例是否会导致调试器挂起多次。这个新选项如图 32 所示。

[![A screenshot of the new debug option for recurring exception instances.](img/92563474f9fc16d1d5d25cce24a5b076.png "img_5e6f36f9726f0")](/sites/default/files/blog/2020/03/img_5e6f36f9726f0.png)

Figure 32\. A new workspace preference for exception breakpoints.

当调试一个在体系结构的几个层次上都有`try`块的应用程序时，您会发现这个选项很有用。在这种情况下，对于同一个异常实例，异常断点可能会触发多次。例如，`catch`块中的`throw`语句可能会再次抛出相同的异常。

对于每个`finally`块或`try-with-resources`块也是如此。当调试器由于异常断点而停止时，您可能希望通过按下 **Resume (F8)** 来继续您的调试会话，但是所有的捕捉和重新抛出将迫使您观察相同异常将反复出现的所有位置。

暂停调用堆栈上的所有`try`块还会破坏打开 Java 编辑器的上下文，因为它会打开更多与手头的调试任务无关的类的编辑器。JDT 调试器现在可以检测到这种情况。它第一次注意到表面上重复出现的同一个异常实例时，您会看到图 33 中的问题对话框。

[![A screenshot of the JDT Debugger inquiring about a recurring exception instance.](img/b396fb0b3a711ba8597924f8e89a6738.png "img_5e6f37130799d")](/sites/default/files/blog/2020/03/img_5e6f37130799d.png)

Figure 33\. The JDT Debugger spots a recurring exception instance.

如果在此对话框中选择 **Skip** ，异常实例将被永久删除。如果您不做任何事情，相同异常类型的新实例将在抛出时导致挂起。然而，如果您选择**记住我的决定**，您的选择将被存储在您的工作区首选项中，并应用于所有异常断点。即使在首选项中只选择了**跳过—重复**一次，您也可以通过简单地按下**单步返回(F7)** 而不是**恢复**来恢复旧的行为。

### JDT 开发人员的内容帮助

三个现有的扩展点现在允许一个新属性`requiresUIThread`，它标记内容辅助扩展是否需要在 UI 线程中运行。扩展点是:

*   `org.eclipse.jdt.ui.javaCompletionProposalComputer`
*   `org.eclipse.jdt.ui.javadocCompletionProposalComputer`
*   `org.eclipse.jdt.ui.javaCompletionProposalSorters`

开发者可以使用`requiresUIThread`属性来声明是否需要在 UI 线程中运行。内容辅助操作将使用此信息进行优化，并通过减少 UI 线程中的工作量来防止 UI 冻结。为了保持向后兼容性，该属性的默认值(如果没有设置)是`true`，这意味着扩展应该在 UI 线程中运行。

## 结论

JBoss Tools 的最新版本改进了基于容器的开发，支持 OpenShift 容器平台 4.3 和新的 OpenShift 应用程序浏览器视图。我们为 Quarkus 添加了工具，更新了 Hibernate 工具运行时提供者，并更新了 Java 开发工具(JDT)扩展以兼容 Java 13。

在本文中，我分享了 Hibernate 工具和 JDT 扩展的变化要点，以及 JBoss Tools 4.14.0 中对平台视图、对话框和工具栏的许多较小的升级和特性添加。现在我们已经发布了 JBoss Tools 4.14.0 和 Red Hat CodeReady Studio 12.14，我们已经在为 Eclipse 2020-03 的下一个版本工作了。请继续关注即将发布的版本信息。

*Last updated: June 29, 2020*
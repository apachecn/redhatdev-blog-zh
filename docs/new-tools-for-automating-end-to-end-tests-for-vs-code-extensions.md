# 用于自动化 VS 代码扩展的端到端测试的新工具

> 原文：<https://developers.redhat.com/blog/2019/11/18/new-tools-for-automating-end-to-end-tests-for-vs-code-extensions>

在发布软件之前，从用户的角度测试软件是一种常见的做法。带着这个假设，我已经着手寻找一个带有自动化端到端测试的 [VS 代码](https://developers.redhat.com/blog/category/vs-code/)扩展。我的探索以失败告终。很自然地，像我这样一个懒惰的人会问:“为什么没有人尝试自动化呢？”事实证明，这种自动化实际上是相当困难的。

然后，我的任务变成了找到一个解决方案，让开发人员能够做到这一点。我很高兴地宣布，没有更多的时间需要浪费在这个卑微的，手工活动。进入名副其实的 [`vscode-extension-tester`](https://www.npmjs.com/package/vscode-extension-tester) :一个让你为你的 VS 代码扩展创建自动化测试并轻松启动它们的框架。你只需要一个 [`npm`包](https://www.npmjs.com/package/vscode-extension-tester)。

## 它是如何工作的

VS 代码基于[电子](https://electronjs.org/)，所以是 web 应用。因此，我们的想法是使用 [Selenium WebDriver](https://www.seleniumhq.org/docs/03_webdriver.jsp) 来自动化测试。为此，我们需要:

1.  1.  下载适当版本的 ChromeDriver，这意味着要知道我们的 VS 代码使用的电子浏览器中的 Chromium 版本。
    2.  将 ChromeDriver 添加到我们的路径中。
    3.  选择合适的 VS 代码二进制(每个操作系统都不同)。
    4.  设置我们的 VS 代码来正确运行测试。例如，我们不能使用原生标题栏。
    5.  下载另一个 VS 代码实例只是为了测试。(我们不想弄乱我们实际使用的 VS 代码的实例。)
    6.  建造我们的延伸部分。
    7.  将扩展安装到新实例中。

    最后，我们已经准备好开始编写我们的测试，但是图 1 显示了为了按下按钮并打开一个视图，我们必须筛选的内容:

    [![](img/96feb53f2688bb8c64b02889da5b83fe.png "vscodeDom")](/sites/default/files/blog/2019/10/Screenshot-from-2019-10-30-09-24-42.png)

    Figure 1: The mass of VS Code DOM contents we would have to sift through.

    为了找到一个代表视图容器的图标，需要 15 层 block 元素，这对于找到一个简单的元素来说是一个很高的要求。你可以想象 DOM 的其他部分是什么样子。但足够的恐吓战术，我们在这里使测试令人兴奋。几乎和编码本身一样令人兴奋，因为我们正在把测试变成编码。让我们看看一旦我们采用了`vscode-extension-tester`框架，这一切变得多么容易。

    ## 化繁为简

    为了进行演示，我们将采用一个扩展，并使用我们的框架为它创建端到端测试。作为第一步，我喜欢使用一些简单的东西，比如来自微软的[扩展示例回购](https://github.com/microsoft/vscode-extension-samples)的 [`helloworld`示例扩展](https://github.com/microsoft/vscode-extension-samples/tree/master/helloworld-sample)。这个扩展贡献了一个名为`Hello World`的新命令，它显示了一个通知说`Hello World!`现在我们需要编写测试来验证这个命令是否正常工作。

    [![](img/7b40c263f83cdbe43aa1d80638901b73.png "demo")](/sites/default/files/blog/2019/10/demo.gif)

    Figure 2: The Hello World command: Enlarge to play.

    ### 获取依赖关系

    首先，我们需要获得必要的依赖关系。首先，我们需要扩展测试器本身，以及它集成的测试框架: [Mocha](https://www.npmjs.com/package/mocha) 。我们可以从`npm`注册表中获得这两者:

    ```
    $ npm install --save-dev vscode-extension-tester mocha @types/mocha
    ```

    我还会用[柴](https://www.npmjs.com/package/chai)做断言。您可以使用任何您喜欢的断言库:

    ```
    $ npm install --save-dev chai @types/chai
    ```

    ### 设置测试

    现在我们已经安装了依赖项，我们可以开始把所有的部分放在一起。让我们从创建一个测试文件开始。我们的测试文件将保存在`src/ui-test` 文件夹中，但是您可以使用您的 tsconfig 包含的任何路径，因为我们将像扩展的其余部分一样用 TypeScript 编写我们的测试。让我们继续创建我们选择的文件夹，并在其中创建一个测试文件。我会叫我的`helloworld-test.ts`。我们的文件结构现在应该如图 3 所示:

    [![](img/93b7eefa0228e4f4dbd7b516aa6ab745.png "files")](/sites/default/files/blog/2019/10/files.png)

    Figure 3: Our beginning file structure.

    接下来，我们需要一种方法来启动我们的测试。为此，我们在我们的`package.json`文件中创建了一个新脚本。让我们称我们的新脚本为`ui-test`*，并使用扩展测试器附带的 [CLI](https://github.com/redhat-developer/vscode-extension-tester/wiki/Test-Setup#using-the-cli) ，称为`extest`。在这个演示中，我们希望使用默认配置和最新版本的 VS 代码、默认设置和默认存储位置(我们稍后将回到这里)。*

     *我们还希望执行所有的设置，然后在一个命令中运行我们的测试。为此，我们可以使用 [`setup-and-run`](https://github.com/redhat-developer/vscode-extension-tester/wiki/Test-Setup#set-up-and-run-tests) 命令，该命令以 [glob](https://www.npmjs.com/package/glob) 的形式将测试文件的路径作为参数。注意，我们不能使用原始的`.ts`文件来启动测试。相反，我们需要使用编译过的`.js`文件，在本例中，它位于`out/`文件夹中。该脚本将看起来像这样:

    ```
    "ui-test": "extest setup-and-run out/ui-test/*.js"
    ```

    在尝试运行测试之前编译测试也很重要，我们可以和其余的代码一起编译测试。为此，这个扩展有一个我们可以使用的编译脚本。最终的脚本将如下所示:

    ```
    "ui-test": "npm run compile && extest setup-and-run out/ui-test/*.js"
    ```

    ### 设置构件

    现在是时候谈谈我前面提到的存储文件夹的重要性了。这是框架存储测试运行所需的一切的地方，包括 VS 代码的一个新实例，ChromeDriver 二进制文件，可能还有失败测试的截图。必须将该文件夹排除在编译和`vsce`打包之外。否则，您肯定会遇到构建错误。我们还建议将存储文件夹添加到您的`.gitignore`文件中。默认情况下，这个文件夹名为`test-resources`，创建在您的扩展库的根目录下。

    首先，让我们从编译中排除该文件夹。我们需要打开`tsconfig.json`文件，并将存储文件夹添加到`"exclude"`数组中。这是我的 tsconfig 现在的样子:

    ```
    {
    	"compilerOptions": {
    		"module": "commonjs",
    		"target": "es6",
    		"outDir": "out",
    		"sourceMap": true,
    		"strict": true,
    		"rootDir": "src"
    	},
    	"exclude": ["node_modules", ".vscode-test", "test-resources"]
    }
    ```

    有了这些代码，我们的扩展就不会在文件夹存在的情况下遇到构建错误。接下来，我们需要确保在打包扩展时，这个文件夹没有包含在最终的`.vsix`文件中。为此，我们可以利用`.vscodeignore`文件。如果它还不存在，让我们继续在我们的库的根中创建一个。然后，将文件夹放入其中，就像我们使用`.gitignore`一样，如图 4 所示:

    [![](img/d44e14ff644c074951cca6f7bd6780d2.png "vsignore")](/sites/default/files/blog/2019/10/vsignore.png)

    Figure 4: Excluding the test-resources directory from packaging.

    完成这三个简单的步骤后，我们就可以开始编写测试了。如果您希望获得关于测试设置的更多信息，请查看框架的 [wiki](https://github.com/redhat-developer/vscode-extension-tester/wiki/Test-Setup) 。

    ## 编写测试

    还记得那个来自 VS Code DOM 的可怕截图吗？如果你熟悉 WebDriver 测试，你就会知道当元素结构如此复杂时，测试会变得多么乏味。

    ### 介绍页面对象

    幸运的是，我们现在不需要为 DOM 而烦恼。Extension Tester 框架为我们带来了全面的[页面对象 API](https://github.com/redhat-developer/vscode-extension-tester/wiki/Page-Object-APIs) 。

    VS 代码中的每种类型的组件都由一个特定的 typescript 类来表示，并且可以通过一组易于理解的方法来操作。我们建议浏览一下[页面对象快速指南](https://github.com/redhat-developer/vscode-extension-tester/wiki/Page-Object-APIs)，以了解每个对象在浏览器中代表什么。此外，每个对象都扩展了普通 WebDriver 的 WebElement，因此您可以根据自己的需要使用普通的 WebDriver 代码。

    ### 回到手头的测试

    首先，我们需要使用 [Mocha BDD](https://mochajs.org/#bdd) 格式创建一个测试套件和一个测试用例。我们测试用例的第一步是执行命令`Hello World`。为此，我们可以使用`Workbench`类及其`executeCommand`方法。我们的测试文件现在看起来有点像这样:

    ```
    import { Workbench } from 'vscode-extension-tester';

    describe('Hello World Example UI Tests', () => {
        it('Command shows a notification with the correct text', async () => {
            const workbench = new Workbench();
            await workbench.executeCommand('Hello World');
        });
    });

    ```

    很简单，不是吗？现在，我们需要断言正确的通知已经出现。这个命令需要时间来执行和显示结果，所以我们不能马上断言。因此，我们使用 WebDriver 来等待通知的出现。为此，我们需要一个合适的等待条件。

    我们的等待条件需要查看当前显示的通知，并返回符合我们需要的通知。在这种情况下，通知将包含文本`Hello`。如果没有找到这样的条件，则不返回任何内容(返回未定义)。这样，一旦返回第一个真值，等待就会终止:

    ```
    async function notificationExists(text: string): Promise<Notification | undefined> {
        const notifications = await new Workbench().getNotifications();
        for (const notification of notifications) {
            const message = await notification.getMessage();
            if (message.indexOf(text) >= 0) {
                return notification;
            }
        }
    }
    ```

    有了这个条件，我们现在开始等待。为此，我们需要一个对底层 WebDriver 实例的引用。我们可以从`VSBrowser`对象获得引用，它是扩展测试 API 的入口点。我们将使用`before`函数在测试运行之前初始化 WebDriver 实例，方法是在套件的开头添加以下代码行:

    ```
        let driver: WebDriver;

        before(() => {
            driver = VSBrowser.instance.driver;
        });
    ```

    启动等待现在就像这样简单:

    ```
    const notification = await driver.wait(() => { return notificationExists('Hello'); }, 2000) as Notification;
    ```

    注意结尾的演员阵容。我们的等待条件可能会返回 undefined，我们需要使用一个`Notification`对象。

    最后一步是通过检查通知是否有正确的文本来断言我们的通知有正确的属性，并且是一个`info`类型。用柴的`expect`来完成这个任务看起来是这样的:

    ```
            expect(await notification.getMessage()).equals('Hello World!');
            expect(await notification.getType()).equals(NotificationType.Info);
    ```

    至此，我们的第一个测试完成了。整个测试文件应该如下所示:

    ```
    import { Workbench, Notification, WebDriver, VSBrowser, NotificationType } from 'vscode-extension-tester';
    import { expect } from 'chai';

    describe('Hello World Example UI Tests', () => {
        let driver: WebDriver;

        before(() => {
            driver = VSBrowser.instance.driver;
        });

        it('Command shows a notification with the correct text', async () => {
            const workbench = new Workbench();
            await workbench.executeCommand('Hello World');
            const notification = await driver.wait(() => { return notificationExists('Hello'); }, 2000) as Notification;

            expect(await notification.getMessage()).equals('Hello World!');
            expect(await notification.getType()).equals(NotificationType.Info);
        });
    });

    async function notificationExists(text: string): Promise<Notification | undefined> {
        const notifications = await new Workbench().getNotifications();
        for (const notification of notifications) {
            const message = await notification.getMessage();
            if (message.indexOf(text) >= 0) {
                return notification;
            }
        }
    }
    ```

    ## 启动测试

    现在剩下的就是开始我们的测试了。为此，我们可以使用我们最喜欢的终端，启动我们在设置阶段创建的脚本:

    ```
    $ npm run ui-test
    ```

    现在我们可以看到工具自动为我们运行设置:

    [video width = " 1920 " height = " 1080 " webm = " https://developers . red hat . com/blog/WP-content/uploads/2019/11/extest _ screen cast . webm "][/video]

    我们的测试运行是成功的:我们验证了我们的扩展的特性是有效的。最重要的是，我们不再需要手动完成所有这些工作。

    ## 了解更多信息

    如果你想了解更多关于使用扩展测试器的知识，请务必访问 [GitHub 知识库](https://github.com/redhat-developer/vscode-extension-tester)或 [npm 注册页面](https://www.npmjs.com/package/vscode-extension-tester)。尤其是[的维基](https://github.com/redhat-developer/vscode-extension-tester/wiki)，可能会令人感兴趣。

    要查找本文中所有步骤的详细描述，请参见下面的链接:

    *   [测试设置](https://github.com/redhat-developer/vscode-extension-tester/wiki/Test-Setup)
    *   [示例测试用例](https://github.com/redhat-developer/vscode-extension-tester/wiki/Writing-Simple-Tests)
    *   [页面对象 API 快速指南](https://github.com/redhat-developer/vscode-extension-tester/wiki/Page-Object-APIs)

    对我们在本文中使用的示例项目感兴趣吗？在[示例项目](https://github.com/redhat-developer/vscode-extension-tester/tree/master/sample-projects)部分查看它的代码，包括注释测试。

    我们也已经有了一些真正扩展的工作测试套件(不仅仅是示例套件)。请随意寻找灵感:

    *   Apache Camel 扩展测试套件。
    *   融合工具的 [UI 测试工具](https://github.com/djelinek/vscode-uitests-tooling)，延伸延伸测试仪。
    *   [Extension Tester 自己的测试套件](https://github.com/redhat-developer/vscode-extension-tester/tree/master/test/test-project)，几乎覆盖了所有可用的页面对象。

    如果你想参与，请查看[贡献者指南](https://github.com/redhat-developer/vscode-extension-tester/blob/master/CONTRIBUTING.md)。我们总是很高兴看到你的反馈和建议，或者你的代码贡献。* 

**Last updated: July 1, 2020**
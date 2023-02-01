# 用 Espresso 测试你的 Android 应用的用户界面

> 原文：<https://developers.redhat.com/blog/2017/07/13/testing-your-android-apps-ui-with-espresso>

Android 是市场上使用最多的移动操作系统之一，估计市场份额约为 84.82%。Android 操作系统中有数以百万计的应用程序，用于各种任务，但遗憾的是，只有一小部分应用程序拥有完善的用户界面(UI)，该界面灵活，可适应各种手机尺寸。对于普通用户来说，他们希望他们的应用程序看起来不错，做得很好。然而，如果你是一名应用程序开发者，你将面临一个巨大的问题，Android 是开源的，它出现在各种屏幕尺寸的各种手机中。Android 开发人员已经知道了这个问题，并引入了一个新的自动化测试框架来测试你的应用程序的 UI，名为 Espresso。

如上所述，Espresso 是 Android 推出的一个自动化测试框架，用于测试你的应用程序的用户界面，并确保它在所有尺寸怪异的设备上都能正常运行。因此，我将向您展示如何一步一步地启动和运行测试您的应用程序活动。我们实际上将有 2 篇博客文章，一篇作为开始，另一篇作为第二部分，支持 CI 平台上的多种屏幕尺寸，以减少博客的长度。所以让我们先来设置浓缩咖啡。

## 设置 Espresso

设置 Espresso 很简单，就像 JUnit test runner(已经添加)一样，我们将 Espresso 添加到我们应用程序的“build.gradle”文件的依赖项部分下:

```
dependencies {
    // Other dependencies ...
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:2.2.2'
    androidTestImplementation 'com.android.support.test:runner:0.5'
}
```

*注意:大多数安卓和谷歌的网站都指定使用“androidTestCompile ”,但我使用了“androidTestImplementation ”,这是因为谷歌在其 2017 年 I/O 中透露,“Compile”组[“compile”、“androidTestCompile”等。]将在不久的将来折旧，并由“实施”组取代。*

好了，既然我们已经包括了 Espresso，让我们深入研究如何为 Espresso 编写一些测试，记住我不会在这里设计任何 UI。我只是在解释，我们将如何编写 Espresso 测试。

## 编写浓缩咖啡测试

因此，你进入你的应用程序中的 Espresso tests 目录[/app/src/java/com/example/Android/testing/Espresso]并创建一个新的 Java 类。

然后，您可以指定测试应该使用' @RunWith()'运行的所有 UI，并指定要运行的 JUnit 测试。然后像对所有其他 Java 文件一样指定类名，然后用“@Rule”指定规则，用“@Before”指定测试运行前要做的事情。最后，用“@Test”指定测试。

您需要用要测试的活动来初始化活动测试规则，并指定相应的 Java 文件。

您将初始化由“@Before”表示的函数中的变量。函数名可以是任何东西。

然后，我们将在带有“@Test”标签的函数中推出我们的测试。

我们现在可以开始测试了。我们可以测试几乎每一个 UI 元素，只要它有一个有效的唯一“id”集。我们这样做，通过使用 espresso 中提供的 onView 函数，我们提供一个由另一个名为“withId()”的函数生成的整数，我们将“Id”作为参数提供给该函数，如“R.id.blah”。然后我们指定我们需要执行什么动作。Espresso 提供了类似'的子功能。执行(click())'，'。perform(typeText(字符串文本))'。我们还可以检查这些测试是否符合。check()'子函数。

以下是 Android 开发者网站提供的一个示例测试:

```
package com.example.android.testing.espresso.BasicSample;

import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;
...

@RunWith(AndroidJUnit4.class)
@LargeTest
public class ChangeTextBehaviorTest {

    private String mStringToBetyped;

    @Rule
    public ActivityTestRule&lt;MainActivity&gt; mActivityRule = new ActivityTestRule&lt;&gt;(
            MainActivity.class);

    @Before
    public void initValidString() {
        // Specify a valid string.
        mStringToBetyped = "Espresso";
    }

    @Test
    public void changeText_sameActivity() {
        // Type text and then press the button.
        onView(withId(R.id.editTextUserInput))
                .perform(typeText(mStringToBetyped), closeSoftKeyboard());
        onView(withId(R.id.changeTextBt)).perform(click());

        // Check that the text was changed.
        onView(withId(R.id.textToBeChanged))
                .check(matches(withText(mStringToBetyped)));
    }
}
```

然后你坐下来，用虚拟机或安卓手机运行你的测试。在下一篇教程中，我们将深入探讨如何测试所有这些奇怪的屏幕尺寸。

总结一下，现在，希望你们都玩得开心...
快乐编码:)

* * *

**红帽移动应用平台** [**下载**](https://developers.redhat.com/products/mobileplatform/download/) **，可在** [**红帽移动应用平台**](https://developers.redhat.com/products/mobileplatform/overview/) **了解更多。**

*Last updated: July 12, 2017*
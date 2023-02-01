# 第 2 部分:构建无服务器服务

> 原文：<https://developers.redhat.com/coderland/serverless/building-a-serverless-service>

现在[我们已经解释了场景](/coderland/serverless/serverless-knative-intro "Part 1: Introduction to Serverless with Knative")，让我们看看服务本身。我们将介绍图像处理代码及其工作原理，我们将对其进行测试，然后我们将在[唐·申克](https://developers.redhat.com/blog/author/donschenck/)的 React 前端中使用该代码。

如果你更多的是一个观察者而不是读者，这里有这个内容的视频版本:

[https://www.youtube.com/embed/M_Xse7vjkvE?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/M_Xse7vjkvE?autoplay=0&start=0&rel=0)

## **图像操作代码**

图像处理代码是编译驱动程序 Photo Booth 的核心。我们把高清网络摄像头钉在柱子上，贴在游乐设施旁边:

![webcam](img/e0fc148f92f0bb59c76751a58e4b4fea.png)

当骑车人经过摄像头时，摄像头会拍下兴奋的公园游客的照片，并将数据发送到 Coderland Swag 商店。Swag Shop 将数据发送给 Knative，后者调用服务并返回结果。然后，客人可以购买修改过的照片的副本。

代码本身是一个 SpringBoot 应用程序，我们马上就会看到。SpringBoot 使我们能够轻松地构建 web 应用程序。由于摄像机发送的数据量很大，我们需要使用一个`POST`请求来从服务中获取数据。SpringBoot 通过定义处理`POST`请求及其端点和数据格式的方法的注释使这变得容易。我们将使用其他注释来处理数据整理和其他必要的事情。

无服务器代码通常使用 JSON 作为其输入和输出格式。因为我们在这里处理的显然是二进制数据，所以我们必须对数据进行 64 位编码和解码。另一个架构问题是，我们需要在 Java 应用程序中使用 JSON。为此，我们将使用 Jackson JSON 库，并设置一个助手类，让我们完全忽略任何与解析或创建 JSON 数据相关的问题。

## **构建和运行代码**

我们部署到 Knative 的服务是一个 SpringBoot 应用程序，前端是一个需要 Node.js 的 React 应用程序。一旦构建了代码，测试它的最简单方法是通过命令行上的`curl`。(您可以用其他方式测试代码，但是它们更复杂。)在构建和运行代码之前，您需要安装 Java、Maven、Node 和`curl`。假设你已经有了这些东西，我们将在分析代码之前运行它。

当然，第一步是克隆回购协议:

![Github code](img/f0537c48bb7b26086ce2bf026abafc76.png)*Github Code**[Image Overlay Code](https://github.com/redhat-developer-demos/image-overlay)* *接下来，我们需要构建代码。这很简单:只需切换到`image-overlay`目录并运行以下命令:

```
​​​​​​​mvn clean package
```

Maven 构建代码，运行一些测试，并将所有东西打包成一个 JAR 文件放在`target`目录中。构建好 JAR 文件后，用`java`命令运行它:

```
java -jar target/imageOverlay-1.0.0.jar
```

这将启动`localhost:8080/overlayImage`上的代码。现在该跑`curl`了。切换到另一个终端提示符，准备输入一些内容，因为我们需要在`curl`命令上指定很多内容:

*   这是一个`POST`请求

*   我们正在发送 JSON ( `ContentType: application/json`)

*   我们只接受 JSON 的响应(`Accept: application/json`)

*   我们正在发送一个包含在回购(`sampleInput.txt`)中的数据文件

*   服务的 URL，`localhost:8080/overlayImage`

此外，您可能希望将命令的输出重定向到一个文件，以便能够处理结果，所以在末尾添加类似于 `> results.txt`的内容。振作起来；下面是完整的命令:

```
curl -X "POST" -H "Content-Type: application/json" -H "Accept: application/json" -d @sampleInput.txt localhost:8080/overlayImage > results.txt
```

对于这里的选项，`X`指定了 HTTP 请求的类型，`H`定义了 HTTP 头，`d`定义了输入数据，at 符号表示这是一个文件名。

所以运行`curl`命令，然后编辑`results.txt`。你需要做的是删除除了`imageData`之外的所有内容。(只保留双引号之间的内容。)完成后，应该只剩下 imageData 的值了。没有双引号，没有字段名，没有花括号。通过将文件另存为`results.txt`来覆盖该文件。

最后，运行`base64`解码图像:

```
base64 -D results.txt -o results.jpg
```

这将解码`results.txt`文件，并将转换的输出写入文件`results.jpg`。如果您对 JSON 数据感到满意，编辑`sampleInput.txt`，更改文件末尾的`greeting`字段，再次经历这个过程，您将看到一个顶部带有不同问候语的新图像。

## **服务的代码**

既然您已经知道了如何构建和运行代码，我们就来看看它是如何工作的。下面是该应用程序的基本流程:

*   使用各种 SpringBoot 注释来设置应用程序。

*   从`POST`请求中获取数据。同样，我们使用杰克逊图书馆为我们做到这一点。

*   从数据中创建一个`BufferedImage`，然后为它创建一个画布(一个`Graphics2D`对象)。

*   将`BufferedImage`绘制到画布上。

*   获得 Coderland 徽标。它作为资源存储在应用程序的 JAR 文件中，因此很容易找到和加载。

*   一旦我们有了标志，我们就把它画在画布上原始图像的上面。

*   从 JSON 数据中获取消息，并使用`Font`和`FontMetrics`对象来计算消息需要多少空间。

*   在画布中央绘制消息文本。

*   获取日期戳，算出它需要多少空间，并把它画在画布的中央。

*   将画布上完成的图像转换为二进制数据。

*   将二进制数据转换为 64 进制

*   将数据放入 JSON 结构并返回给调用者。

(你可能会争辩说，任何有那么多条目的列表都不应该被称为“基本流程”。这是一个很好的观点，但是代码真的没有那么复杂。如你所见。)

这里是我们使用的第一个注释。这是来自`com.example.imageOverlay.ImageOverlayApplication.java`的代码:

```
@SpringBootApplication
public class ImageOverlayApplication {

 @RestController
 class ImageOverlayController {
```

我们将其定义为`@SpringBootApplication`，并说`ImageOverlayController`类是一个`@RestController`。这使得我们可以轻松地说，我们的代码处理特定端点的`POST`请求:

```
​​​​​​​   @PostMapping(path = "/overlayImage",
                consumes = "application/json",
                produces = "application/json")
   public Image incomingImage(@RequestBody Image image)
     throws IOException {
```

这里我们说方法`incomingImage`在`/overlayImage`端点处理`POST`请求。正如我们之前讨论的，它使用 JSON 作为输入和输出格式。我们使用`@RequestBody`注释来说明一个`Image`对象(接下来会详细介绍这个类)是`POST`请求的主体。

在进入实际的图像处理之前，我们先来看看`Image`类。它有六个成员字段:

*   `imageData` -图像的基本 64 位编码数据

*   `imageType`-JPG 或巴布亚新几内亚，不区分大小写

*   `greeting` -写在图像顶部的消息

*   `language` -日期字符串使用的语言(例如，`en`表示英语)

*   `country` -日期字符串使用的国家(`US`表示美国)

*   `dateFormatString` -定义日期格式的字符串。“`MMMM d, yyyy`”是默认值。

前三个字段是最有用的，而后三个只是我们移入 JSON 数据的硬编码参数。将最后三个字段更改为`pl`、`PL`和`d MMMM yyyy`，将生成字符串“17 lutego 2019”(*顺便给我们波兰的朋友们。)你更有可能改变`greeting`字段。*

**注意:**我们知道`example.com.imageOverlay.Image`是一个很糟糕的名字，我们对此感到很糟糕。但还不至于糟糕到让我们花时间去想一个更好的名字。

这些是`Image`类的成员字段。但是为什么首先要上课呢？因为我们可以用它来使用 Jackson JSON 库。当我们用`@JsonProperty`注释设置`Image`类时，Jackson 自动将我们的 JSON 输入转换成 Java 对象，然后自动将我们的 Java 对象转换成我们需要的 JSON 输出。我们不需要解析任何东西。下面是对`imageData`字段的注释:

```
@JsonProperty("imageData")
   public String getImageData() {
       return imageData;
   }

   public void setImageData(String imageData) {
       this.imageData = imageData;
   }
```

这告诉 Jackson 库，当解析 JSON 数据以创建 Image 对象时，JSON 字段`imageData`映射到 Java 类中的成员字段`imageData`。当我们要求 Jackson 将 Java 对象转换回 JSON 时，同样的映射反向工作。(名字不一定要一样，如果一样的话就少了很多混淆。)

这为处理 JSON 数据和处理来自网络的`POST`请求奠定了基础。现在是时候进入实际的图像处理代码了。我们会很快过一遍。了解如何构建 REST 应用程序和处理 JSON 数据是您将在很多地方使用的有用技能；处理图像，可能就没那么简单了。我们从从`imageData`创建一个`BufferedImage`对象开始:

```
BufferedImage baseImage =
       ImageIO.read(Base64.getDecoder().
                    wrap(new StringBufferInputStream(imageData)));

     int imageTypeCode = imageType.equalsIgnoreCase("png") ?
       BufferedImage.TYPE_INT_ARGB : BufferedImage.TYPE_INT_RGB;

     BufferedImage targetImage =
       new BufferedImage(baseImage.getWidth(),
                         baseImage.getHeight(),
                         imageTypeCode);

     Graphics2D canvas = (Graphics2D) targetImage.getGraphics();
     canvas.drawImage(baseImage, 0, 0, null);
     AlphaComposite alphaChannel =
       AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 1.0f);
     canvas.setComposite(alphaChannel);
```

一旦我们有了`BufferedImage`，我们必须查看`imageType`字段，看看这个图像是否有透明的 alpha 通道。(png 有一个，JPEGs 没有。)接下来我们创造`targetImage`。这是输出图像。我们得到这个图像的画布(一个`Graphics2D`对象)并在空白画布上绘制原始图像(`baseImage`)。现在我们准备在原始图像上绘制东西，所以我们设置了 alpha 通道。

从这里，我们加载包含 Coderland 徽标的覆盖图像:

```
BufferedImage logoImage =
       ImageIO.read(ImageOverlayApplication.class.
               getResourceAsStream("/static/images/overlay.png"));
```

映像的位置基于类路径，类路径是在 Maven 生成的 JAR 文件中设置的。文件在 repo 中的实际位置是`src/main/resources/static/images/overlay.png`，但是您在 Java 中使用的路径的根是从`static`目录开始的。无论如何，如果您想用别的东西替换 Coderland 徽标，您可以替换那个图像，重新构建 JAR 并运行它。

在目标图像上绘制徽标就像调用`drawImage()`方法一样简单。但是，我们做了一点数学计算，以确保徽标在图像上垂直居中，不管它的尺寸是多少:

```
int centerX = 0;
     int centerY = (baseImage.getHeight() -
                    (int) logoImage.getHeight()) / 2;
     canvas.drawImage(logoImage, centerX, centerY, null);
```

现在是时候绘制问候和日期戳的文本了。为此，我们将设置画布的字体，然后使用`Font`和`FontMetrics`对象来精确计算文本需要多少空间。这样，我们可以在图像上水平居中文本。作为最后一步，我们将黑色文本向下画两个像素，向右画两个像素，然后在黑色文本上画白色文本。这给了我们一个很好的阴影效果，如果照片的背景比较浅，文字会更容易阅读。代码如下:

```
canvas.setFont(new Font(Font.SANS_SERIF, Font.BOLD, 48));
     FontMetrics fontMetrics = canvas.getFontMetrics();
     Rectangle2D rect =
       fontMetrics.getStringBounds(greeting, canvas);

     centerX = (baseImage.getWidth() - (int) rect.getWidth()) / 2;
     centerY = 100; 

     canvas.setColor(Color.BLACK);
     canvas.drawString(greeting, centerX + 2, centerY + 2);
     canvas.setColor(Color.WHITE);
     canvas.drawString(greeting, centerX, centerY);
```

这里有一些改进的机会:

*   我最初写这段代码是为了使用 transition，Red Hat 的官方字体。如果代码找不到该字体，它就使用`Font.SANS_SERIF`作为后备。然而，当您将服务部署到 Knative 时，您将它部署为一个容器映像。这意味着容器图像必须包含天桥字体。如果你知道如何修改 repo 中的`Dockerfile`来给图像添加立交桥，我们很想看看你是怎么做的。(如果你给我们发一份公关，我们*真的*会很高兴。)

*   无论图像的大小，文本的边距和偏移量以及徽标的大小都不会改变。那样会更优雅。

*   正如您在上面的清单中看到的，我们将字体大小设置为 48。这对于某些图像或问候来说可能太大了。例如，如果问候语是葛底斯堡演说的前几行，那就不合适:

![doug-coderland](img/e3a2d86edfd62ef531af02a7c17d05f7.png)

如果我得到文本的宽度并缩小字体大小，直到所请求的文本适合图像，那就太聪明了。我们称之为读者练习。(你可能会称之为程序员的懒惰。)继续，下一步是使用 JSON 数据中指定的地区(`language`和`location`)和`dateFormatString`获取日期:

```
SimpleDateFormat sdf =
       new SimpleDateFormat(dateFormatString,
                            new Locale(language, location));
     String dateString = sdf.format(new Date());
```

一旦我们有了日期字符串，我们就把它画在图像的底部，就像我们在顶部画问候一样。

最后，我们有了更新的图像。Coderland 徽标、问候和日期戳都覆盖在图像上，所以是时候创建一些 JSON 并发送回去了。为此，我们将数据放在一个`ByteArrayOutputStream`中，使用 Java 的`Base64`实用程序类对其进行编码，然后创建一个新的`Image`对象。(那是我们表示 JSON 数据的`Image`类。正如我们所说，这是一个糟糕的名字。)

最后，我们返回`Image`对象。

```
ByteArrayOutputStream overlaidImage =
       new ByteArrayOutputStream();
     ImageIO.write(targetImage, imageType, overlaidImage);
     overlaidImageData = (Base64.getEncoder().
                encodeToString(overlaidImage.toByteArray()));

     Image updatedImage =
       new Image(overlaidImageData, "JPG", greeting,
                 language, location, dateFormatString);
     return updatedImage;
```

注意，在整个过程中，没有看到 JSON 数据。Jackson 在处理所有细节方面做得很好，让我们专注于图像处理，而不是 JSON 解析。

现在，您知道了如何构建和运行代码，并且了解了它是如何工作的。毫无疑问，您还注意到，向代码发送一些数据、分解响应并查看生成的图像是多么的繁琐。幸运的是，我们已经有了一个非常非常简单的方法来测试代码。

## **反应前端**

当我在整理这些材料时，我和令人敬畏的唐·申克(Don Schenck)聊了聊，他有了一个好主意。(为了使事情不那么冗长，我将使用“Don”作为“The Awesome etc”的缩写。)为什么不创建一个 React web 应用程序来访问您机器上的网络摄像头，然后每当您单击一个按钮时就将图像从网络摄像头发送到服务？当然，这是一个很棒的想法，我们很幸运唐主动提出为我们编写 web 应用程序。

首先，切换到另一个终端窗口，克隆 Don 的 repo:

![Github code](img/f0537c48bb7b26086ce2bf026abafc76.png)*Github Code**[React front-end for the Image Overlay code](https://github.com/redhat-developer-demos/coderland-photo-store)* *接下来键入`npm install`和`npm start`来设置和运行 Don 的代码。代码应该会在系统的默认浏览器中打开一个选项卡。如果没有，就去`localhost:3000`。你会看到一个类似这样的显示:

![photo-booth1](img/e7d47864499e517b6abd9d71f210634e.png)

正如你从上面光线不佳的图片中看到的，浏览器通过我笔记本电脑的网络摄像头捕捉我的图像。(对不起，太晚了，我不想给你拍一张漂亮的照片。)确保图像处理服务正在运行，单击“拍照”按钮，您将在页面底部看到转换后的图像:

![](img/0fc562466dae3108c45a3bfb428db409.png)

从网络摄像头捕获的图像已经过修改，现在显示在屏幕底部。(你也可以明白我之前说的根据图像的大小来改变页边距和文本大小是什么意思。)

可悲的是，你的作者没有资格讨论《唐的密码》的奇迹。当与 Don 交谈时，他带着特有的谦逊说代码没什么大不了的，这是基本的 React 和`nodefetch`以及`react-webcam`库。我相信任何有 React 技能的人都会发现代码很简单。希望 Don 很快会有时间更详细地解释他的代码。我们会很高兴用那段内容替换这一段。

## 下一步是什么

显而易见的下一步是继续第三部 [。在那篇文章中，您将获取代码并将其部署到 Knative](/coderland/serverless/deploying-serverless-knative "Part 3: Deploying a Serverless Service to Knative") 。然后您将看到如何在 OpenShift 控制台中监控服务，如何从命令行调用它，最后如何将 Don 的前端与 Knative 管理的无服务器代码一起使用。

我们是否提到过，当您向我们发送您的评论和问题时，它会带给我们快乐？请通过[coderland@redhat.com](mailto:coderland@redhat.com)联系我们，我们希望收到您的来信。

*Last updated: April 21, 2021***
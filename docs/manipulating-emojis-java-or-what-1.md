# 在 Java 中操纵表情符号，或者:什么是？+ 1?

> 原文：<https://developers.redhat.com/articles/2019/08/16/manipulating-emojis-java-or-what-1>

警告:你将要看到的代码没有任何可取之处。我们希望你和我们一样喜欢它。

如果你和我们一样，你可能想知道如何在你的 Java 程序中操作表情符号。或者你可能一直在思考那个古老的问题，“什么是？+ 1?"多亏了最近的一次编码会议，你的作者花了几个小时完成了一个？？(兔子洞)，我们可以帮你回答这个问题。

首先，这里是完整的代码，`MysteryAnimal.java`:

```
public class MysteryAnimal {
  public static void main(String[] args) {
    String bear = "?";

    // If the previous line doesn't show up in your editor,
    // you can comment it out and use this declaration instead: 
    // String bear = "\ud83d\udc3b";

    int bearCodepoint = bear.codePointAt(bear.offsetByCodePoints(0, 0));
    int mysteryAnimalCodepoint = bearCodepoint + 1;

    char mysteryAnimal[] = {Character.highSurrogate(mysteryAnimalCodepoint),
                            Character.lowSurrogate(mysteryAnimalCodepoint)};
    System.out.println("The Coderland Zoo's latest attraction: "
                       + String.valueOf(mysteryAnimal));
  }
}
```

代码创建了一个 Java `String`,它的值是熊表情符号。请注意，您可以用字符串`"\ud83d\udc3b"`来表示熊表情符号，这是两个成对工作的 Unicode 值。定义了字符串(bear =？)，我们获取表情符号的 Unicode 码位，给它加 1，将更新后的码位转换回一对新值，最后打印结果。

Java 的问题是一个`char`只能代表一个 4 位十六进制数(16 位)，而表情符号是 5 位十六进制数。如果你有时间消磨(坦白地说，如果你正在读这篇文章，你可能会这样做)，Unicode 网站有[表情符号](https://unicode.org/emoji/charts/full-emoji-list.html)的完整列表。剧透:看看这份名单可能会给出这个谜题的答案。 小熊表情符号的 Unicode 码位是`\u1f43b`。同样，这个数字太大了，无法用一个`char`来表示。

Unicode 协会的解决方案是创建所谓的*代理字符*。(尽管表情符号对现代人类的存在至关重要。)表情符号被表示为一对有序的 4 位十六进制数字:高代理和低代理。有相当复杂的数学公式来根据原始的 5 位十六进制数计算这两个值，但是 Java `Character`类，正如您在上面看到的，可以为您计算这两个值。为了在 Java `String`中表示表情符号，我们用两个代理创建了一个`char`数组，并使用`String.valueOf()`方法将它们组合起来。

你可以在 Java `Character`类的文档中找到更多关于代理字符的信息。你可以在 Unicode 标准本身的中找到更多关于代理的信息。

## 现在，答案来了

如果您运行上面的代码，您将得到以下结果:

```
The Coderland Zoo's latest attraction: ?
```

现在你知道了！从表情符号的角度来看，熊+ 1 =熊猫。我们希望你喜欢这种转移；如果你已经读到这里，我们假设你可能已经读过了。享受在 Java 中操作表情符号的乐趣，如果你发现这项技术有任何实际用途，请在评论中告诉我们。玩得开心！

*Last updated: May 24, 2021*
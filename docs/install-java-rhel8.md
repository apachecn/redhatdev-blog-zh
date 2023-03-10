# 如何在红帽企业版 Linux 8 上安装 Java 8 和 11

> 原文：<https://developers.redhat.com/blog/2018/12/10/install-java-rhel8>

有了红帽企业 Linux (RHEL) 8，将支持两个主要版本的[Java](/topics/enterprise-java):Java 8 和 Java 11。在本文中，我将把 Java 8 称为 JDK (Java 开发工具包)8，因为我们关注的是使用 Java 的开发方面。JDK 8 和 JDK 11 分别指 OpenJDK 8 和 OpenJDK 11 的红帽版本。

通过这篇文章，您将了解如何在 RHEL 8 上安装和运行简单的 Java 应用程序，如何通过`alternatives`在两个并行安装的主要 JDK 版本之间切换，以及如何基于每个应用程序从两个 JDK 中选择一个。

## TL；速度三角形定位法(dead reckoning)

要安装 JDK 8，请使用:(如果您在安装过程中没有选择*让这个用户成为管理员*，请参见本文的[在 RHEL](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用 sudo)

```
$ sudo yum install java-1.8.0-openjdk-devel
```

然后运行 Java“Hello World”，如下所示:

```
$ cat > HelloWorld.java <<HELLO
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World!");
  }
}
HELLO
$ javac HelloWorld.java && java HelloWorld
Hello World!
```

要安装 JDK 11，请使用:

```
$ sudo yum install java-11-openjdk-devel
```

然后运行 Java“Hello World”，如下所示:

```
$ cat > HelloWorld.java <<HELLO
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World!");
  }
}
HELLO
$ /usr/lib/jvm/java-11-openjdk/bin/java HelloWorld.java
Hello World!
```

是的，有了 JDK 11，你可以直接运行 Java 源文件。编译步骤已为您处理。

## 视频演示

如果你喜欢看 4 分钟的简短演示视频，这里是:

[https://www.youtube.com/embed/0TMbIMb4pOA?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/0TMbIMb4pOA?autoplay=0&start=0&rel=0)

## 更长的版本...

让我们假设我们有一台新委托的 Red Hat Enterprise Linux 8 机器，我们想用它来运行 Java 应用程序。

## 查找可用的 Java 包

为了确定要安装哪些 RPM 包，我们可以询问打包系统哪些包提供了`java`二进制文件:

```
$ yum provides \*/bin/java

```

这个命令告诉我们包`java-1.8.0-openjdk-headless`和`java-11-openjdk-headless`都提供了`java`二进制文件。出于本文的目的，我们对开发包感兴趣，所以我们将安装-devel 子包。-devel 包将把无头包作为一个依赖项拉进来。如果您已经知道 RHEL 包是 OpenJDK 版本，`yum list available`可能也很有用:

```
$ yum list available \*openjdk\*

```

出于本文的目的，我们打算并行安装 JDK 8 和 JDK 11**，还要安装 maven:**

```
$ sudo yum install java-1.8.0-openjdk-devel java-11-openjdk-devel maven

```

## 切换 Java 版本

在上一步中，我们并行安装了 JDK 8 和 JDK 11。此时，JDK 8 号是 RHEL 8 号上的主要 JDK。这就是为什么在新安装的 RHEL 8 上运行`java -version`时会得到这个输出:

```
$ java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)

```

有两种方法可以选择您想要的 Java 版本:

1.  通过替换选项在系统范围内切换`java`和`javac`二进制文件。这种方法**需要 root 权限**。
2.  通过设置 JAVA_HOME 为每个应用程序选择 JDK

### 使用替代选项选择 Java 版本

RHEL 8 号上的`java`和`javac`二进制文件由替代系统管理。这意味着系统管理员可以将系统`java`(或`javac`)切换为不同于默认的 JDK 8。替代系统使用优先级来确定哪个 JDK 应该通过`/usr/bin/java`可用。JDK 8 号在 RHEL 8 号上的优先权高于 JDK 11 号。但是我们太超前了。首先，让我们看看哪些二进制文件是由替代品管理的:

```
$ alternatives --list

```

我们看到`java`和`javac`是由替代管理的。接下来，我们将使用`alternatives --config`命令切换到 JDK 11 号:

```
$ sudo alternatives --config java
There are 2 programs which provide 'java'.
Selection      Command
-----------------------------------------------
*+ 1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-8.el8.x86_64/jre/bin/java)
   2           java-11-openjdk.x86_64 (/usr/lib/jvm/java-11-openjdk-11.0.1.13-4.el8.x86_64/bin/java)

Enter to keep the current selection[+], or type selection number: 2

```

这将把系统`java`二进制转换到 JDK 11。我们对`javac`也是如此，因为`java`和`javac`是独立管理的。没有必要切换其他任何东西，因为每一个其他的 JDK 二进制将与`java`或`javac`二进制切换:

```
$ sudo alternatives --config javac
There are 2 programs which provide 'javac'.
Selection      Command
-----------------------------------------------
*+ 1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-8.el8.x86_64/jre/bin/javac)
   2           java-11-openjdk.x86_64 (/usr/lib/jvm/java-11-openjdk-11.0.1.13-4.el8.x86_64/bin/javac)

Enter to keep the current selection[+], or type selection number: 2

```

### 非交互切换选项

这里有一种方法，通过替代方案，使用非交互方式切换到 JDK 11，如果您需要编写脚本，这很方便:

```
$ JAVA_11=$(alternatives --display java | grep 'family java-11-openjdk' | cut -d' ' -f1)
$ sudo alternatives --set java $JAVA_11

```

同样，通过非交互方式的替代方案切换到 JDK 8:

```
$ JAVA_8=$(alternatives --display java | grep 'family java-1.8.0-openjdk' | cut -d' ' -f1)
$ sudo alternatives --set java $JAVA_8

```

对于`javac`，可以遵循类似的方法。

### 通过设置 JAVA_HOME 选择 Java 版本

许多应用程序支持使用`JAVA_HOME`环境变量，作为指定应该使用哪个 JDK 来运行应用程序的一种方式。下面的例子演示了在运行 maven 时的这种用法。如果您没有 root 权限，但是两个 JDK 都已经安装在您的系统上，那么这种方法非常方便。

选择 JDK 8:

```
$ JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk mvn --version
Apache Maven 3.5.4 (Red Hat 3.5.4-5)
Maven home: /usr/share/maven
Java version: 1.8.0_191, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-9.el8.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-2.el8.x86_64", arch: "amd64", family: "unix"
```

选择 JDK 11:

```
$ JAVA_HOME=/usr/lib/jvm/java-11-openjdk mvn --version
Apache Maven 3.5.4 (Red Hat 3.5.4-5)
Maven home: /usr/share/maven
Java version: 11.0.1, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-11-openjdk-11.0.1.13-6.el8.x86_64
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-2.el8.x86_64", arch: "amd64", family: "unix"
```

如果你安装了多个次要的 JDK 版本，这个特性也很方便，正如 Leo Ufimtsev 在他的文章中对 RHEL 7 的描述。

[![](img/c2b9f06305c4b924feb10978643812c5.png)](https://che.openshift.io/f?url=https://raw.githubusercontent.com/redhat-developer/devfile/master/getting-started/java-maven/devfile.yaml/?sc_cid=7013a000002D1quAAC)

## 你好，类固醇世界，JDK 11 风格

有了 JDK 11 的一部分 JEP 330 T1，现在有可能以类似脚本的方式运行 Java 代码。这个特性叫做“启动单文件源代码程序”,允许用户使用 Java 作为脚本语言。这里有一个简单的例子:

```
$ cat > factorial <<FACT
#!/usr/lib/jvm/java-11-openjdk/bin/java --source 8
public class Factorial {

  private static void usage() {
    System.err.println("factorial <number>");
    System.exit(1);
  }

  private static long factorial(int num) {
    if (num <= 1) {
      return 1;
    }
    if (num == 2) {
      return 2;
    }
    return factorial(num - 1) * num;
  }

  public static void main(String[] args) {
    if (args.length != 1) {
      usage();
    }
    int num = -1;
    try {
      num = Integer.parseInt(args[0]);
    } catch (Exception e) {
      System.err.println("Error: Argument not a number!");
      usage();
    }
    System.out.println(factorial(num));
  }
}
FACT
$ chmod +x factorial
$ ./factorial 6
720
```

谢谢你。我希望这篇文章对你有用。

*Last updated: September 29, 2022***
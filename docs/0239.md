# 通过 6 个简单的步骤在生产 Red Hat OpenShift 容器中使用 vim

> 原文：<https://developers.redhat.com/blog/2021/01/19/use-vim-in-a-production-red-hat-openshift-container-in-6-easy-steps>

**免责声明**:大多数情况下，我们不建议在容器中编辑文件。但是，在极少数情况下，您可能需要在生产容器中复制并稍微修改文件，尤其是在调试时。(在本例中，我使用的 [vim](https://fedoraproject.org/wiki/Vim) 方法适用于我笔记本电脑上的 Fedora 32，它是我的 [Red Hat OpenShift](http://developers.redhat.com/openshift) 容器映像的基础。)

在本文中，当 vim 没有安装在[容器](https://developers.redhat.com/topics/containers/)映像中时，我演示了如何在生产 Red Hat OpenShift 容器中安装和运行 vim。我还描述了用于克服本地操作系统和容器基础映像不一致的方法。

### 步骤 1:复制 vim 二进制文件

为了让`oc cp`工作，复制 vim 二进制文件:

 ````
$ cp /usr/bin/vim ~/Downloads/vim
```

(这种复制 vim 二进制文件的方式最适合我，尽管可能有另一种更干净的方式。如果你有不同的方法，请在评论中告诉我。)

### 步骤 2:登录到`oc`集群

要登录到`oc`集群，请运行以下命令:

```
$ oc login ..
```

### 步骤 3:指定要安装 vim 的容器

我的 pod 中只有一个容器在运行，所以 oc 会自动选择 pod 中的第一个容器:

```
$ export POD=yourPodName
```

### 步骤 4:复制运行 vim 所需的文件

为了让 vim 正常启动，请复制这个文件列表。如果 vim 没有启动，将文件添加到该列表并复制它们:

```
$ export VIM_DEPS="~/Downloads/vim /lib64/libgpm.so.2.1.0 /lib64/libpython3.8.so.1.0 /lib64/libgpm.so.2"
```

```
$ for i in $VIM_DEPS; do oc cp $i $POD:/home/worker; done
```

### 第 5 步:登录 pod

要登录到 pod，请运行以下命令:

`$ oc rsh $POD`

### 步骤 6:运行 vim

要运行 vim，请输入以下内容:

```
worker@pod$ LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/worker" PATH="$PATH:$PWD" vim
```

### 如果我的操作系统不同于容器的基本映像，该怎么办？

确保您的容器映像和笔记本电脑的架构相匹配。然后启动一个可以在本地容器中运行的基本映像。在本地主机上运行的容器中安装 vim。照原样复制出 vim 二进制文件；例如，使用`podman cp`或`docker cp`，并如前所述将其复制到 pod。在 pod 中运行 vim，观察哪些文件丢失了。这些文件可以从本地主机上运行的容器中获取。

当您需要在 OpenShift 生产容器中复制和稍微修改文件时，我希望这个快速提示 vim 能有所帮助。

*Last updated: January 28, 2021*`
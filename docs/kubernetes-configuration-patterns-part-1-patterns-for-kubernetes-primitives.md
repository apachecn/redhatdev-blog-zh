# Kubernetes 配置模式，第 1 部分:Kubernetes 原语的模式

> 原文：<https://developers.redhat.com/blog/2021/04/28/kubernetes-configuration-patterns-part-1-patterns-for-kubernetes-primitives>

本文是关于 Kubernetes 配置模式的两部分系列文章的第一部分，它代表了配置 Kubernetes 应用程序和控制器的方法。第 1 部分介绍了只使用 Kubernetes 原语的简单方法。这些模式适用于在 Kubernetes 上运行的任何应用程序。[第 2 部分介绍了更高级的模式](/blog/2021/05/05/kubernetes-configuration-patterns-part-2-patterns-for-kubernetes-controllers)。当您开发 Kubernetes 控制器时，这些模式要求您针对 Kubernetes API 进行编码。

**注意** : *Kubernetes 控制器*是运行在 Kubernetes 上并管理 Kubernetes 中资源的程序。

为了简单起见，我在示例 YAML 文件中只使用了`Deployment` s。然而，这些示例应该与其他的`PodSpecable`(任何描述`PodSpec`的东西)一起工作，比如`DaemonSet`和`ReplicaSet` s。我还省略了像`image`、`imagePullPolicy`这样的字段，以及示例`Deployment` YAML 中的其他字段。

## 带命令行参数的配置

Kubernetes 允许您向容器传递命令行参数，这有助于为应用程序提供配置:

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            containers:
            - name: server
              args:
              - "gravity=10"
              - "colorMode=dark"

```

命令行参数`gravity`和`colorMode`将被传递给在容器中运行的应用程序。应用程序需要读取命令行参数并应用它们。

这个 Kubernetes 配置模式不是很灵活:配置是在 YAML 中硬编码的，所以配置是耦合到`Deployment`的。此外，当在多个`Deployment`中需要相同的配置时，这种模式不能很好地扩展。

这种实践可以用于将使用命令行参数进行配置的遗留软件迁移到容器中。然而，从灵活性和可伸缩性的角度来看，本文中介绍的任何替代方案都会更好。

## 环境变量配置

Kubernetes 允许您在容器中设置环境变量。容器中的应用程序可以使用这些环境变量作为配置选项。例如:

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
    spec:
        template:
            spec:
                containers:
                - name: server
                env:
                - name: GRAVITY
                  value: "10"
                - name: COLOR_MODE
                  value: "dark"

```

环境变量`GRAVITY`和`COLOR_MODE`将被传递给在容器中运行的应用程序。应用程序需要读取环境变量并应用它们。当环境变量发生变化时，将推出`Deployment`。

这种模式几乎与前一种模式相同，即带有命令行参数的[配置，因为该配置在`Deployment` YAML 中仍然是硬编码的。然而，与命令行参数相比，环境变量更容易定义并提供给容器。此外，在命令行参数方法中，运行应用程序的配置和命令在同一个地方。](#basics_1)

您很快就会看到，Kubernetes 还允许您传递环境变量来获取不同来源的值，比如`Secret` s、`ConfigMap` s 和引用。

如果您不需要在不同的`Deployment`或容器之间共享配置，这种模式提供了最简单和最直接的配置方法。

## 反模式:带有 NFS 卷挂载的配置

使用 Kubernetes 可以将网络驱动器安装到容器上。

在这个模式中，想法是将一个包含配置文件的卷挂载到一个容器中。稍后，在容器中运行的应用程序可以从文件系统中读取配置文件。

要创建这样的卷，我们可以使用网络文件系统(NFS)卷，如图 1 所示。

[![This Kubernetes configuration pattern performs configuration with an NFS volume mount.](img/f7f4a4ac88b942528fcb4f5c3999f2ae.png "basics_3")](/sites/default/files/blog/2020/11/basics_3.png)

Figure 1: You can mount a volume with a configuration into a container.

下面是用于实现挂载的`Deployment`的一部分:

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            volumes:
            - name: nfs-volume
              nfs:
                  server: nfs.example.com
                  path: /configs/game-server/
            containers:
            - name: server
              volumeMounts:
              - name: nfs-volume
                mountPath: /etc/config

```

在这个例子中，`game-server`容器将能够读取`/etc/config/`目录下的文件，该目录将是在`Deployment`中定义的 NFS 服务器中的`/configs/game-server/`路径的反映。

在 Kubernetes 上创建`Deployment`之前，您需要在 NFS 服务器上手动创建配置文件。

这种模式非常类似于以文件模式挂载 ConfigMap 的[配置，我将很快展示这一点。这里的区别在于，您需要使用 NFS 并在 NFS 服务器上手动创建配置文件。由于这些需求，这个解决方案是一个反模式，尽管在某些情况下它是有用的。其中一种情况是配置非常大的时候，比如](#basics_5)[机器学习](/topics/ai-ml)模型(如果可以认为是配置的一部分的话)。另一个有用的情况是配置必须是不可变的。

## 反模式:使用初始化容器的配置

Kubernetes 有一个 *init 容器*的概念。这些容器在主容器之前运行，并为它们配置环境。您可以利用这个特性来处理应用程序的配置。

在这种模式中，配置位于由 init 容器运行的独立映像中。该容器将其保存的配置复制到与其他容器共享的卷中，如图 2 所示。init 容器保存配置，稍后将配置复制到与服务器容器共享的卷上。

[![In this Kubernetes configuration pattern, an init container copies a configuration to the main container through a shared volume mounted in the main container.](img/f231574c5862da36e4b18f307995fa45.png "basics_4")](/sites/default/files/blog/2020/11/basics_4.png)

Figure 2: An init container can copy a configuration to the main container.

下面是用于实现 init 容器的`Deployment`的一部分:

```
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            volumes:
            - name: config-directory
              emptyDir: {}
            initContainers:
            - name: init
              image: my.image-repository.com/initcontainer
              volumeMounts:
              - name: config-directory
                mountPath: /etc/config
            containers:
            - name: server
              image: my.image-repository.com/maincontainer
              volumeMounts:
              - name: config-directory
                mountPath: /etc/config

```

但是，您需要以这种方式构建 init 容器。例如:

```
FROM busybox
ADD configuration.properties /config-src/configuration.properties
ENTRYPOINT [ "sh", "-c", "cp /config-src/* /etc/config", "--" ]

```

`config.properties`文件是在构建过程中添加到容器映像的配置文件。稍后，用`cp`命令简单地复制该文件。因此，在 init 容器的构建期间，配置应该是可用的。

共享卷中的服务器容器对配置文件所做的任何更改都是暂时的。此外，在构建期间使配置可用不是很灵活，也不是 [12 因素式的](https://12factor.net/config)。

当配置文件太大或者目标是不可变的配置时，您可以在类似于带有 NFS 卷挂载模式的[配置的情况下使用这种模式。但是使用这里显示的 init 容器被认为是反模式，除了这些特定的情况。](#basics_3)

## 将配置图作为文件挂载的配置

可以将 Kubernetes `ConfigMap`作为文件安装到容器中，如图 3 所示。

[![In this Kubernetes configuration pattern, a ConfigMap is mounted as a file in a volume.](img/9118b4b6dba9da779fd131f4f4ba937c.png "basics_5")](/sites/default/files/blog/2020/11/basics_5.png)

Figure 3: A ConfigMap is mounted as a file in a volume.

以下是`Deployment`用于安装`ConfigMap`的部分:

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: game-config
data:
    game.properties: |
        gravity=10
        colorMode=dark
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            containers:
            - name: server
              volumeMounts:
              - mountPath: /etc/config
                name: game-config-volume
            volumes:
            - name: game-config-volume
              configMap:
                  name: game-config

```

`ConfigMap`中的数据将作为卷中的文件安装到`Deployment`中。数据字段中的关键字`game.properties`被用作文件名，该关键字的值将是文件内容。配置文件路径将是`/etc/config/game.properties`。当你按照这个过程，游戏应用程序可以读取和处理`.properties`文件。

这种配置模式易于理解和管理。配置可以作为应用程序容器中的一个完整文件来读取，这样可能会更简单、更快。此外，与将`ConfigMap`的值设置为[环境变量](#basics_2)相比，相同的`ConfigMap`可以很容易地安装到不同的容器中。

使用这种模式，配置与应用程序是松散耦合的。无需触摸`Deployment`即可更改配置。可以在多个应用中使用相同的配置，也可以将多个`ConfigMap`安装到单个`Deployment`中。多个`ConfigMap`在需要时提供更多的粒度。

然而，这种模式需要在容器中运行的应用程序中进行文件读取操作，这可能并不是在所有情况下都是理想的。此外，当使用这种模式时，`Deployment`正在使用的配置在为`Deployment`创建的`Deployment`或`Pod`上是不可见的。操作员需要检查`ConfigMap`来查看配置值。

当有很多配置选项时，`ConfigMap`挂载是有意义的，因为配置是作为一个整体用单个`volumeMount`来提供的。此外，当使用特定的文件格式时，比如前面例子中的`.properties`文件，文件内容可以提供给属性文件解析器，这在许多语言中都可用。

默认情况下，当安装的`ConfigMap`发生变化时，`Deployment`不会重启。有重启`Deployment`的技巧，但是它们超出了本文的范围。

当没有很多配置选项时，您可以使用其他模式，比如从 ConfigMaps 中设置环境变量[(如下所示),以避免文件读取操作和卷装载。](#basics_7)

您可以将挂载的`ConfigMap`设为只读，但不能设为不可变。集群管理员将能够更改`ConfigMap`，从而更改应用程序的配置。如果需要不变性，使用带有 NFS 卷挂载的[配置或带有 init containers](#basics_3) 模式的[配置。](#basics_4)

## 配置图作为多个文件挂载的配置

当一个`ConfigMap`包含多个密钥时，可以将这些密钥作为单独的文件安装。

这种模式与将 ConfigMap 挂载为文件模式的[配置相同，只是它使用多个文件来分隔配置的各个部分，从而创建不同的作用域。图 4 显示了这种模式。](#basics_5)

[![This Kubernetes configuration pattern mounts different ConfigMaps as multiple files in a volume.](img/26ca9434dcda9398358fe883562f0e74.png "basics_6")](/sites/default/files/blog/2020/11/basics_6.png)

Figure 4: ConfigMaps are mounted as multiple files in a volume.

以下是用于创建`ConfigMap`和卷的`Deployment`部分:

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: game-config
data:
    game.properties: |
        gravity=10
    ui.properties: |
        colorMode=dark
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            containers:
            - name: server
              volumeMounts:
              - mountPath: /etc/config
                name: game-config-volume
            volumes:
            - name: game-config-volume
              configMap:
                  name: game-config

```

在配置图作为文件模式挂载的[配置中，`ConfigMap`中的数据将作为卷中的文件挂载到`Deployment`中。不过，这一次将有两个文件:`/etc/config/game.properties`和`/etc/config/ui.properties`。](#basics_5)

这种分离在某些情况下很有用，比如当应用程序模块独立读取它们的配置时，或者当配置有如此多的选项，以至于有必要将选项分成多个逻辑部分，同时将所有内容保存在单个`ConfigMap`中。

## 将配置图用作环境变量来源的配置

当没有太多配置选项并且不需要 I/O 操作时，这种模式是首选。图 5 显示了这种模式。

[![This Kubernetes configuration pattern uses a ConfigMap as a source for environment variables.](img/c911acc8e9ff376f4f9122ff0c420d6f.png "basics_7")](/sites/default/files/blog/2020/11/basics_7.png)

Figure 5: A ConfigMap serves as a source for environment variables.

下面是用于实现`ConfigMap`和环境变量的`Deployment`的一部分:

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: game-config
data:
    gravity: "10"
    colorMode: "dark"
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            containers:
            - name: server
              env:
              - name: GRAVITY
                valueFrom:
                    configMapKeyRef:
                        name: game-config
                        key: gravity
              - name: COLOR_MODE
                valueFrom:
                    configMapKeyRef:
                        name: game-config
                        key: colorMode

```

环境变量将被设置为来自`ConfigMap`的值，最终为`GRAVITY=10`和`COLOR_MODE=dark`。容器中运行的应用程序需要读取环境变量，并将它们用作配置。

这种模式提供了一种将`ConfigMap`中的选项绑定到应用程序中实际使用的配置选项的好方法。在`ConfigMap`中可能有更多的选项，但是并不是每个应用程序都需要所有的选项。

从`ConfigMap`键到环境变量的映射是显式的，并且使这些绑定可读。此外，应用程序不需要任何文件 I/O 操作。

然而，如果有太多的配置选项需要从一个`ConfigMap`绑定到环境变量，事情会变得混乱。在这种情况下，最好使用以文件模式挂载 ConfigMap 的[配置。](#basics_5)

## 带秘密的配置

任何为配置而做的事情理论上都可以用`Secret` s 来完成。将凭证、证书和类似的东西存储在`Secret`中比存储在`ConfigMap`中更好，因为`Secret` s 更安全。

例如，要将`Secret`作为文件挂载到卷上，可以使用以下 YAML:

```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
    name: game-secret
data:
    databaseUsername: Zm9v
    databasePassword: YmFy
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: game-server-deployment
spec:
    ...
    template:
        spec:
            containers:
            - name: server
              volumeMounts:
              - mountPath: /etc/config
                name: game-secret-volume
            volumes:
            - name: game-secret-volume
              configMap:
                  name: game-secret

```

这将创建两个对服务器容器可用的文件:`/etc/config/databaseUsername`和`/etc/config/databasePassword`。

如本例所示，简单地用`Secret`替换`ConfigMap`在大多数情况下应该是可行的。

## 第一部分的结论

本文描述的 Kubernetes 配置模式使用 Kubernetes 原语，将帮助您配置运行在 Kubernetes 上的应用程序。它们实现起来相对简单。本文的第二部分介绍了特定于 Kubernetes 控制器的模式。这些将需要针对 Kubernetes API 进行编码。

*Last updated: October 14, 2022*
# 在 Red Hat OpenShift 容器平台上使用带 AMQ 的箭筒

> 原文：<https://developers.redhat.com/blog/2019/04/24/using-quiver-with-amq-on-red-hat-openshift-container-platform>

作为红帽 [UKI 专业服务](https://www.redhat.com/en/services/consulting)团队的一员，我曾与几个正在[红帽 OpenShift 容器平台(OCP)](https://developers.redhat.com/products/openshift/overview/) 上实现 [AMQ 代理](https://developers.redhat.com/products/amq/overview/)的客户合作过。客户通常会问的一个问题是，“我们如何验证 AMQ 配置对于我们的场景是正确的？”以前，我会提出以下建议之一:

*   [ActiveMQ Perf Maven 插件](http://activemq.apache.org/activemq-performance-module-users-manual.html)
*   [JMeter](http://activemq.apache.org/jmeter-performance-tests.html)
*   [加特林](https://gatling.io/docs/3.0/jms/)

这些工具可以为您提供以下指标:

*   代理启动并运行了吗？也就是说，它能接收/发布该配置的消息吗？
*   经纪人能处理某个性能特征吗？也就是说，对于这种配置，我的最小每秒发布速率是多少？
*   还有更多。

这些工具的问题是您不能选择客户端技术。这可能导致现实世界的差异和有限的技术选择，从而可能导致您走上错误的技术道路。换句话说:

*   与生产中使用的 [AMQ 客户端](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html/amq_clients_overview/components)相比，JMeter 的性能是否相同？你是在进行同类比较吗？苹果配苹果？

那么，我认为答案是什么？[颤](https://github.com/ssorj/quiver)[【1】](#DISCLAIMER)。在这篇文章中，我将提供一个在 Red Hat OpenShift 上使用红帽 AMQ 箭筒的概述和演示。如果你想了解更多关于 AMQ 红帽公司的信息以及它如何帮助你，请查看这个[网上研讨会。](https://youtu.be/mkqVxWZfGfI)

## 颤是什么？

直接来自[颤 GitHub 页面](https://github.com/ssorj/quiver):

> 颤实现是各种语言和 API 的本地客户机(有时也是服务器),它们发送或接收消息，并将有关传输的原始信息写入标准输出。它们故意简单。

> arrow-arrow 命令在发送或接收模式下运行单个实现，并捕获其输出。它有定义执行参数、选择实现和报告统计信息的选项。

> 箭图命令启动一对箭图实例，一个发送者和一个接收者，并生成消息的端到端传输的摘要。

用我自己的话说:

> 颤是一个启动发布者和接收者客户端的工具，目的是尽可能快地完成。一旦发布者和接收者完成；会生成一个统计输出，它可以让您深入了解测试的执行情况。

> 客户端基于 Apache Qpid 项目，该项目提供:Java、C++、Python 和 Javascript 实现等等。客户主要针对 [JMS](https://download.oracle.com/otndocs/jcp/jms-2_0-fr-eval-spec/) 和 [AMQP](http://www.amqp.org/resources/download) 用户。

## 录制的演示

感觉很懒，想看一段[asci NEMA](https://asciinema.org/)的录像？

演示将显示以下内容:

*   AMQ 代理和 AMQ 互连部署到在 Minishift 上运行的 OKD
*   使用代理的核心协议通过颤发送和接收消息
*   使用 JMS 客户机通过 AMQP 协议向代理发送和接收消息
*   使用 AMQP 协议上的 JMS 客户机通过颤发送和接收消息，以实现互连

[![asciicast](img/76251a15d558636d15bdc6c4cb6cebbc.png)](https://asciinema.org/a/240989)

## 自己动手演示

想玩颤，AMQ，和 OCP 自己？

这个演示使用了一个 OCP 模板，它需要两个参数:

*   DOCKER _ IMAGE 完全限定的 docker 拉 URL 的位置。这可以通过导入的图像流解决；`oc get is quiver -o jsonpath=‘{.status.dockerImageRepository}’`
*   DOCKER _ CMD 要执行的 JSON 数组格式的颤命令。

### 演示先决条件

假设您有以下情况:

*   OCP CLI 和 web 控制台的基本知识
*   一个运行 [Red Hat OpenShift 集装箱平台【OCP】或](https://developers.redhat.com/products/openshift/overview/) [Minishift](https://github.com/minishift/minishift) 的环境
*   按照本文档安装的 AMQ 代理模板和图像流: [2.1。在 OpenShift 容器平台图像流和应用程序模板上安装 AMQ 代理](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html-single/deploying_amq_broker_on_openshift_container_platform/index#install-deploy-ocp-broker-ocp)
*   AMQ 互连模板和图像流安装如下: [2.1。验证 AMQ 互连模板的可用性](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html/deploying_amq_interconnect_on_openshift_container_platform/preparing-to-deploy-router-ocp)

### 设置

1.  创建项目

    ```
     $ oc new-project quiver
    ```

2.  导入颤动图像

    ```
     $ oc import-image quiver:latest --from=docker.io/ssorj/quiver --confirm
    ```

3.  将视图角色添加到默认服务帐户

    ```
     $ oc policy add-role-to-user view system:serviceaccount:$(oc project -q):default
    ```

4.  部署 AMQ 经纪人

    ```
     $ oc new-app --template=amq-broker-72-basic \
        -e AMQ_PROTOCOL=openwire,amqp,stomp,mqtt \
        -e AMQ_QUEUES=quiver \
        -e AMQ_ADDRESSES=quiver \
        -e AMQ_USER=anonymous \
        -e ADMIN_PASSWORD=password

     $ oc get pods --watch
    ```

5.  部署 AMQ 互联

    ```
     $ oc new-app --template=amq-interconnect-1-basic

     $ oc get pods --watch
    ```

### 向 AMQ 经纪人发送消息

6.  通过[核心协议](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html-single/using_the_amq_core_protocol_jms_client/)

    ```
     $ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
             DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
             DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/activemq-artemis-jms\", \"--impl\", \"activemq-artemis-jms\", \"--verbose\"]" \
             | oc create -f -
    ```

    向代理发送 100 万条消息
7.  查看核心协议 pod

    ```
     $ oc get pods --watch
     $ oc logs -f {insert value of new pod name}

         ---------------------- Sender -----------------------  --------------------- Receiver ----------------------  --------
         Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Lat [ms]
         -----------------------------------------------------  -----------------------------------------------------  --------
              2.3              0           0       73     64.6       2.3              0           0       73     63.8         0
              4.3          3,337       1,667       71    100.0       4.3          2,529       1,261       75     96.6        42
              6.3         24,727      10,684       71    116.0       6.4         25,178      11,223      106    123.1        36
              8.3        113,170      44,177       64    116.7       8.4        116,495      45,658       66    123.9         2
             10.3        221,942      54,359       59    116.8      10.4        227,809      55,491       50    124.0         2
             12.3        327,983      52,994       56    116.8      12.4        332,016      52,077       44    122.3         2
             14.3        428,866      50,391       54    116.8      14.4        433,091      50,512       41    122.5         2
             16.3        540,823      55,923       62    117.1      16.4        547,056      56,926       49    122.6         2
             18.3        645,499      52,312       60    117.1      18.4        648,975      50,934       48    122.6         2
             20.3        761,704      58,073       58    117.1      20.4        768,000      59,453       55    122.7         2
             22.3        873,509      55,875       61    117.2      22.4        874,857      53,348       50    122.7         3
             24.3        914,014      20,242       33    117.2      24.5        923,768      22,728       27    122.7         4
             26.3      1,000,000      42,950       53    117.4      26.5      1,000,000      38,078       45      0.0         2
         --------------------------------------------------------------------------------
         Subject: activemq-artemis-jms //broker-amq-tcp:61616/activemq-artemis-jms (/tmp/quiver-xo6l0q4a)
         Count:                                          1,000,000 messages
         Body size:                                            100 bytes
         Credit window:                                      1,000 messages
         Duration:                                            23.0 s
         Sender rate:                                       43,414 messages/s
         Receiver rate:                                     43,507 messages/s
         End-to-end rate:                                   43,407 messages/s
         Latencies by percentile:
                   0%:        0 ms                90.00%:        4 ms
                  25%:        2 ms                99.00%:       20 ms
                  50%:        2 ms                99.90%:      249 ms
                 100%:      307 ms                99.99%:      290 ms
         --------------------------------------------------------------------------------
    ```

    的输出
8.  通过 AMQP 协议 [JMS 客户端](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html-single/using_the_amq_jms_client/)

    ```
     $ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
             DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
             DOCKER_CMD="[\"quiver\", \"amqp://broker-amq-amqp:5672/qpid-jms-over-ampq\", \"--impl\", \"qpid-jms\", \"--verbose\"]" \
             | oc create -f -
    ```

    向代理发送 100 万条消息
9.  查看 AMQP 客户端 pod 的输出

    ```
     $ oc get pods --watch
     $ oc logs -f {insert value of new pod name}

         ---------------------- Sender -----------------------  --------------------- Receiver ----------------------  --------
         Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Lat [ms]
         -----------------------------------------------------  -----------------------------------------------------  --------
              2.2          2,663       1,330      109    103.1       2.2          1,984         991       94     94.6        89
              4.2         16,304       6,817      118    110.7       4.2         16,113       7,050      115    113.2        56
              6.2         45,720      14,693      102    119.7       6.2         45,809      14,767       97    115.4       118
              8.2        105,097      29,674       88    120.8       8.2         92,810      23,489       84    115.6       305
             10.2        159,018      26,920       71    120.8      10.2        146,602      26,883       76    115.6       411
             12.2        213,794      27,361       81    121.5      12.2        205,564      29,466       81    117.2       400
             14.2        271,138      28,643       77    121.6      14.2        256,132      25,271       76    117.2       461
             16.2        325,058      26,960       82    116.6      16.2        311,757      27,785       75    117.2       566
             18.2        383,747      29,315       78    116.6      18.2        362,224      25,208       72    117.2       678
             20.2        431,554      23,880       64    116.6      20.2        411,578      24,665       70    117.2       750
             22.2        492,444      30,430       75    116.6      22.2        468,113      28,225       77    117.2       786
             24.2        537,684      22,609       70    116.7      24.2        510,287      21,087       70    117.3     1,105
             26.2        595,639      28,963       77    116.7      26.2        567,833      28,759       75    117.4     1,049
             28.2        646,014      25,175       67    116.7      28.2        619,412      25,790       73    117.5     1,014
             30.2        701,401      27,680       73    116.9      30.2        669,677      25,095       72    117.5     1,167
             32.2        757,523      28,019       73    116.9      32.2        722,166      26,231       72    117.6     1,246
             34.2        803,007      22,731       70    117.1      34.2        768,587      23,210       66    117.6     1,360
             36.2        856,193      26,593       69    117.1      36.2        816,627      23,924       73    117.6     1,538
             38.2        911,459      27,605       73    117.1      38.2        870,633      26,990       77    117.6     1,633
             40.2        968,803      28,658       70    117.4      40.2        926,359      27,849       76    117.6     1,482
         ---------------------- Sender -----------------------  --------------------- Receiver ----------------------  --------
         Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Lat [ms]
         -----------------------------------------------------  -----------------------------------------------------  --------
             42.2      1,000,000      15,583       41      0.0      42.2        978,444      26,029       89    117.6     1,581
                -              -           -        -        -      44.2      1,000,000      10,773       57      0.0     1,608
         --------------------------------------------------------------------------------
         Subject: qpid-jms amqp://broker-amq-amqp:5672/qpid-jms-over-ampq (/tmp/quiver-a1xz3nbz)
         Count:                                          1,000,000 messages
         Body size:                                            100 bytes
         Credit window:                                      1,000 messages
         Duration:                                            42.4 s
         Sender rate:                                       24,696 messages/s
         Receiver rate:                                     23,616 messages/s
         End-to-end rate:                                   23,573 messages/s
         Latencies by percentile:
                   0%:        1 ms                90.00%:     1575 ms
                  25%:      534 ms                99.00%:     1741 ms
                  50%:      988 ms                99.90%:     1886 ms
                 100%:     1929 ms                99.99%:     1926 ms
         --------------------------------------------------------------------------------
    ```

### 向 AMQ 互联发送消息

10.  通过 AMQP 协议

    ```
    $ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
            DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
            DOCKER_CMD="[\"quiver\", \"amqp://amq-interconnect:5672/qpid-jms-over-ampq-via-interconnect\", \"--impl\", \"qpid-jms\", \"--verbose\"]" \
            | oc create -f -
    ```

    通过 JMS 客户端向互连发送 1，000，000 条消息
11.  查看 AMQP 客户端 pod

    ```
    $ oc get pods --watch
    $ oc logs -f {insert value of new pod name}

        ---------------------- Sender -----------------------  --------------------- Receiver ----------------------  --------
        Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Lat [ms]
        -----------------------------------------------------  -----------------------------------------------------  --------
             2.2          1,529         764      131    102.6       2.2          1,465         731      120     84.5        21
             4.2         12,580       5,514      151    116.2       4.2         12,529       5,526      138    114.5        16
             6.2         42,121      14,756      125    119.5       6.2         41,815      14,606       90    111.3         6
             8.2         85,563      21,721       82    119.9       8.2         85,028      21,596       71    111.7         4
            10.2        125,149      19,773       75    120.2      10.2        124,858      19,905       62    112.0         5
            12.2        169,899      22,264       79    116.5      12.2        169,965      22,486       67    112.1         5
            14.3        212,816      21,437       69    116.7      14.2        212,442      21,217       60    112.1         5
            16.3        259,889      23,536       75    116.7      16.2        259,571      23,553       71    114.7         4
            18.3        308,674      24,332       86    119.1      18.2        308,723      24,539       69    114.9         4
            20.3        340,342      15,826       67    119.1      20.2        340,176      15,726       58    115.0         6
            22.3        386,804      23,208       76    119.3      22.2        386,597      23,199       67    115.0         4
            24.3        434,611      23,880       75    119.3      24.2        434,435      23,895       65    115.0         4
            26.3        472,270      18,783       63    119.4      26.2        471,754      18,650       53    115.0         6
            28.3        507,116      17,406       59    119.4      28.2        506,646      17,420       51    115.0         6
            30.3        541,841      17,345       57    119.4      30.2        541,639      17,488       56    115.1         6
            32.3        577,421      17,781       61    119.4      32.2        576,429      17,352       50    115.1         6
            34.3        612,512      17,528       61    119.4      34.2        612,231      17,892       53    115.1         6
            36.3        644,913      16,192       58    119.5      36.2        644,190      15,972       48    115.1         6
            38.3        678,782      16,901       58    119.5      38.2        678,577      17,176       52    115.1         7
            40.3        714,240      17,720       60    119.5      40.2        713,569      17,487       51    115.3         6
        ---------------------- Sender -----------------------  --------------------- Receiver ----------------------  --------
        Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Time [s]      Count [m]  Rate [m/s]  CPU [%]  RSS [M]  Lat [ms]
        -----------------------------------------------------  -----------------------------------------------------  --------
            42.3        750,187      17,965       62    119.5      42.2        748,967      17,672       51    115.3         6
            44.3        787,234      18,496       63    119.5      44.3        787,500      19,171       55    115.3         6
            46.3        820,124      16,420       58    119.5      46.3        819,155      15,820       48    115.3         6
            48.3        856,316      18,087       59    119.5      48.3        855,868      18,338       51    115.3         6
            50.3        897,520      20,551       68    119.5      50.3        897,232      20,641       57    115.3         5
            52.3        944,594      23,537       73    119.5      52.3        944,159      23,452       63    115.3         4
            54.3        990,567      22,986       72    119.5      54.3        990,176      22,997       64    115.3         4
            56.3      1,000,000       4,716       17      0.0      56.3      1,000,000       4,907       15      0.0         4
        --------------------------------------------------------------------------------
        Subject: qpid-jms amqp://amq-interconnect:5672/qpid-jms-over-ampq-via-interconnect (/tmp/quiver-0sttx8_z)
        Count:                                          1,000,000 messages
        Body size:                                            100 bytes
        Credit window:                                      1,000 messages
        Duration:                                            53.7 s
        Sender rate:                                       18,615 messages/s
        Receiver rate:                                     18,634 messages/s
        End-to-end rate:                                   18,614 messages/s
        Latencies by percentile:
                  0%:        0 ms                90.00%:       10 ms
                 25%:        3 ms                99.00%:       18 ms
                 50%:        5 ms                99.90%:       43 ms
                100%:       89 ms                99.99%:       72 ms
        --------------------------------------------------------------------------------
    ```

    的输出

### 清除

12.  清理并删除所有箭盒

    ```
    $ oc delete pod -l app=quiver
    ```

## 你的客户使用另一种语言吗？

如果您选择的语言没有在演示中显示，好消息是，颤支持几个[实现](https://github.com/ssorj/quiver#quiver-1)。以下是一些例子:

```
$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/activemq-jms\", \"--impl\", \"activemq-jms\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/qpid-messaging-cpp\", \"--impl\", \"qpid-messaging-cpp\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/qpid-messaging-python\", \"--impl\", \"qpid-messaging-python\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/qpid-proton-c\", \"--impl\", \"qpid-proton-c\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/qpid-proton-cpp\", \"--impl\", \"qpid-proton-cpp\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/qpid-proton-python\", \"--impl\", \"qpid-proton-python\", \"--verbose\"]" \
        | oc create -f -

$ oc process -f https://raw.githubusercontent.com/ssorj/quiver/0.2.0/packaging/openshift/openshift-pod-template.yml \
        DOCKER_IMAGE=$(oc get is quiver -o jsonpath='{.status.dockerImageRepository}'):latest \
        DOCKER_CMD="[\"quiver\", \"//broker-amq-tcp:61616/rhea\", \"--impl\", \"rhea\", \"--verbose\"]" \
        | oc create -f -
```

## 引人深思的事

我希望这个演示已经展示了使用箭筒与 OCP 上的 AMQ 经纪人和 AMQ 互联进行交互是多么容易。

它也应该提出了如何将箭筒集成到您自己的系统中的问题。例如:

*   颤动能成为你的 mvn 集成测试的一部分吗？
*   能否在您的 CI/CD 管道中增加一个烟度测试阶段?。
*   冒烟测试的结果会通过/未通过部署吗？
*   颤动可以作为心跳监控系统的一部分来验证代理是否还活着吗？

这是几个我认为颤动非常有用的场景，你可能还想到了其他场景。

### 注意

[1]尽管箭筒是由 Red Hat 员工开发的，但它不受 Red Hat 订阅的支持，严格来说是一个上游项目。

*Last updated: September 3, 2019*
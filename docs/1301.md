# 为 ActiveMQ 配置 mKahaDB 持久性存储

> 原文：<https://developers.redhat.com/blog/2017/11/22/configure-mkahadb-persistence-storage-activemq-better-management-reduce-disk-usage>

在这篇文章中，我想说明如何在 ActiveMQ 上配置 mKahaDB 持久性存储，以便更好地管理和减少磁盘使用。

当代理管理的所有目的地(队列/主题)具有相似的性能时，默认配置的 *KahaDB* 持久性适配器工作良好。然而，涉及多个第三方的企业解决方案从来都不是这样。

有多个队列或主题以及不同的消费者或收听者在收听这些队列/主题。一些消费者可能比其他消费者慢。这将迅速增加消息存储的磁盘使用量。由于这种情况和单身 *KahaDB* 所有商店的目的地可能会执行缓慢。

*multi KahaDB (mKahaDB)* 持久性适配器允许我们在多个 *KahaDB* 消息存储库中拥有一个代理队列/主题。因此，我们可以将不同的 *KahaDB* 商店分配到不同的队列，从而减少执行速度慢的目的地(因为消费者速度慢)对正常或快速处理目的地的影响。我们可以分配一个单独的 *KahaDB* 商店，在 *mKahaDB* 的帮助下减缓表演目的地的速度。

让我们做一次练习。

1.我们在这次练习中使用了**红帽 JBoss Fuse/AMQ 6.3.0** 。同样应该适用于所有**红帽 JBoss/AMQ 6.x** 版本或者 **ActiveMq 5.x** 版本。
2。代理的配置文件 *activemq.xml* 存在于**红帽 JBoss Fuse/AMQ 6.3.0** 的 etc 文件夹中，我们将对该文件添加或修改配置。
3。让我们将队列的*内存限制*设置为一个更高的值，即 100 MB。以便我们可以向代理发送更多 1mb 大小的消息。

```
<broker xmlns=.... >
---
<destinationPolicy>
<policyMap>
<policyEntries>
<policyEntry topic=">" producerFlowControl="true">
<pendingMessageLimitStrategy>
<constantPendingMessageLimitStrategy limit="1000"/>
</pendingMessageLimitStrategy>
</policyEntry>
<policyEntry queue=">" producerFlowControl="true" memoryLimit="100mb">
</policyEntry>
</policyEntries>
</policyMap>
</destinationPolicy>
---
</broker>
```

注意:我们只配置了*memory limit*=“100 MB”。

4.现在，我们将注释默认的 *KahaDB* 商店配置，并替换为 *mKahaDB* 配置。

```
<broker xmlns=.... >
---
<persistenceAdapter>
<!-- <kahaDB directory="${data}/kahadb"/> -->
<mKahaDB directory="${data}/kahadb">
<filteredPersistenceAdapters>

<filteredKahaDB queue="A.*.C">
<persistenceAdapter>
<kahaDB/>
</persistenceAdapter>
</filteredKahaDB> 

<filteredKahaDB queue="Q.>">
<persistenceAdapter>
<kahaDB/>
</persistenceAdapter>
</filteredKahaDB>

<filteredKahaDB queue="Test.*">
<persistenceAdapter>
<kahaDB/>
</persistenceAdapter>
</filteredKahaDB>

<filteredKahaDB queue="X.Y.Z">
<persistenceAdapter>
<kahaDB/>
</persistenceAdapter>
</filteredKahaDB>

<!-- match all destinations -->
<filteredKahaDB>
<persistenceAdapter>
<kahaDB/>
</persistenceAdapter>
</filteredKahaDB>

</filteredPersistenceAdapters>
</mKahaDB>
</persistenceAdapter>
---
</broker>

```

5.启动代理。

6.现在，如果我们检查位置*JBoss-fuse-6 . 3 . 0 . red hat-310/data/amq/kah ADB*，我们将获得以下目录。

```
[cpandey@cpandey kahadb]$ ls -ltr
total 28
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 07:05 #210
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 07:05 txStore
-rw-rw-r--. 1 cpandey cpandey    8 Nov 18 08:20 lock
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 08:20 queue#3a#2f#2fA.#2a.C
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 08:20 queue#3a#2f#2fQ.#3e
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 08:20 queue#3a#2f#2fTest.#2a
drwxrwxr-x. 2 cpandey cpandey 4096 Nov 18 08:20 queue#3a#2f#2fX.Y.Z
```

**这里要注意的点:**

*   排队。*匹配目标名称前缀队列后的一个单词。
*   排队。>所有以 Queue 为前缀的队列。
*   txStore:这是 mKahaDB 的一个实现细节。它包含涉及不同 KahaDB 实例的事务。
*   #210:这个 KahaDB 商店适用于除了具有特定/过滤的 KahaDB 商店的队列之外的所有人。
*   队列#3a#2f#2fA。#2a。c:这个 KahaDB 存储用于 queue_name 为表达式 A.*的队列。C
*   队列 3a#2f#2fQ。#3e:此 KahaDB 存储用于所有以前缀“q . >”开头的队列。其中'>'代表任何字符串。
*   排队# 3a # 2f #测试。#2a:此队列特定于所有带有表达式测试的队列。*.
*   队列#3a#2f#2fX。Y.Z:该队列专用于 X.Y.Z 队列。

7. *mKahaDB* 是 *KahaDB* 的扩展，因此 *KahaDB* 调谐参数也适用于 *mKahaDB* 。只需确保对于每个滤波器，我们必须应用调整参数，因为每个滤波器都表现为一个单独的 *KahaDB* 存储。更多详情见 [*activemq 文档*](http://activemq.apache.org/kahadb.html#KahaDB-Multi(m)kahaDBPersistenceAdapter) 。

8.我们可以使用 java 代码来发送消息。但是在这里我是用**红帽 JBoss Fuse/AMQ***卡拉夫 终端发送消息。*

**场景 1:** ***KahaDB 存储一个特定的队列。现在让我们使用发送数据到 X.Y.Z 队列。***

#如果我们检查在*kah ADB*store“queue # 3a # 2f # 2fX”中创建的文件。我们看到了日志文件。

```
# At present no messages.
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination X.Y.Z --messageCount 35 --messageSize 1000000
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                       35           0           0          35           0           0           33
JBossFuse:karaf@root
[cpandey@cpandey queue#3a#2f#2fX.Y.Z]$ ls -ltrh
total 34M
-rw-rw-r--. 1 cpandey cpandey 33K Nov 18 09:53 db.redo
-rw-rw-r--. 1 cpandey cpandey 32K Nov 18 09:53 db.data
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 09:53 db-1.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 09:53 db-2.log

 # Now we will purge queue X.Y.Z so that Memory is released, and we can continue test other scenarios.
JBossFuse:karaf@root activemq:purge X.Y.Z
INFO: Purging all messages in queue: X.Y.Z
```

**场景 2: *KahaDB 为队列存储一些常用表达式。在这种情况下，A.*。c 是队列名称的表达式。***

```
# Now let's send data to queue A.B.C.
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination A.B.C --messageCount 35 --messageSize 1000000
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                       35           0           0          35           0           0          33
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0          35          35           0           0
JBossFuse:karaf@root
# Let us check KahaDB store "queue#3a#2f#2fA.#2a.C".
[cpandey@cpandey kahadb]$ cd "queue#3a#2f#2fA.#2a.C"
[cpandey@cpandey queue#3a#2f#2fA.#2a.C]$ ls -ltr
total 34376
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 10:01 db-1.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 10:04 db-2.log
-rw-rw-r--. 1 cpandey cpandey    32824 Nov 18 10:04 db.redo
-rw-rw-r--. 1 cpandey cpandey    32768 Nov 18 10:04 db.data
[cpandey@cpandey queue#3a#2f#2fA.#2a.C]$ 

#Now we will send data to queue A.N.C
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination A.N.C --messageCount 35 --messageSize 1000000
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                       35           0           0          35           0           0          33
A.N.C                                                       35           0           0          35           0           0          33
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0          35          35           0           0
JBossFuse:karaf@root

# If we again observe datastore "queue#3a#2f#2fA.#2a.C", we find that a new journal file is created db-3.log hence content of queue A.N.C is also persisted to this datastore.
[cpandey@cpandey queue#3a#2f#2fA.#2a.C]$ ls -ltr
total 68608
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 10:01 db-1.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 10:08 db-2.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 10:09 db-3.log
-rw-rw-r--. 1 cpandey cpandey    32824 Nov 18 10:09 db.redo
-rw-rw-r--. 1 cpandey cpandey    53248 Nov 18 10:09 db.data
[cpandey@cpandey queue#3a#2f#2fA.#2a.C]$ 

# Now we will purge queue A.B.C and A.N.C so that Memory is released, and we can continue test other scenarios.
JBossFuse:karaf@root activemq:purge A.B.C A.N.C
INFO: Purging all messages in queue: A.N.C
INFO: Purging all messages in queue: A.B.C
```

**场景 3: *KahaDB 存储所有具有相同前缀的队列。这里我们有前缀“Q. >”。***

```
# Sending message to queue Q.1.
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination Q.1 --messageCount 35 --messageSize 1000000
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                         35           0           0          35           0           0          33
Q.1.2                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0           0           0           0           0
JBossFuse:karaf@root
# We see that in folder 'queue#3a#2f#2fQ.#3e' which is journal store for queues with prefix 'Q.' is having db-1.log and db-2.log journals.
[cpandey@cpandey queue#3a#2f#2fQ.#3e]$ ls -ltrh
total 34M
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:37 db-1.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:39 db-2.log
-rw-rw-r--. 1 cpandey cpandey 73K Nov 18 20:39 db.redo
-rw-rw-r--. 1 cpandey cpandey 72K Nov 18 20:39 db.data
[cpandey@cpandey queue#3a#2f#2fQ.#3e]$ 
# Now let us send data to queue Q.1.T and Q.T
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination Q.1.T --messageCount 35 --messageSize 1000000
JBossFuse:karaf@root dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                          0           0           0           0           0           0           0
Q.1                                                         35           0           0          35           0           0          33
Q.1.T                                                       35           0           0          35           0           0          33
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0           0           0           0           0
JBossFuse:karaf@root 
[cpandey@cpandey queue#3a#2f#2fQ.#3e]$ ls -ltrh
total 68M
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:37 db-1.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:41 db-2.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:43 db-3.log
-rw-rw-r--. 1 cpandey cpandey 73K Nov 18 20:43 db.redo
-rw-rw-r--. 1 cpandey cpandey 92K Nov 18 20:43 db.data
JBossFuse:karaf@root activemq:producer --user admin --password admin --destination Q.T --messageCount 35 --messageSize 1000000
[cpandey@cpandey queue#3a#2f#2fQ.#3e]$ ls -ltrh
total 101M
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:37 db-1.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:41 db-2.log
-rw-rw-r--. 1 cpandey cpandey 73K Nov 18 20:43 db.redo
-rw-rw-r--. 1 cpandey cpandey 92K Nov 18 20:43 db.data
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:43 db-3.log
-rw-rw-r--. 1 cpandey cpandey 32M Nov 18 20:43 db-4.log
[cpandey@cpandey queue#3a#2f#2fQ.#3e]$ 
# Above we observe that db journal files has been created in same KahaDB store. Just all queue with prefix 'Q.' 
JBossFuse:karaf@root activemq:purge Q.1 Q.1.T Q.T
INFO: Purging all messages in queue: Q.1
INFO: Purging all messages in queue: Q.1.T
INFO: Purging all messages in queue: Q.T
```

**场景 4: *当消息被发送到过滤器中没有提到的任何其他队列时。我们将在这里观察到，所有未过滤的消息都将转到一个不同的 KahaDB 存储(即全部在#210 内)。***

```
JBossFuse:karaf@root activemq:producer --user admin --password admin --messageCount 35 --messageSize 1000000 --destination FOO
JBossFuse:karaf@root activemq:dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                          0           0           0           0           0           0           0
FOO                                                         35           0           0          35           0           0          33
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.1.T                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0           0           0           0           0
JBossFuse:karaf@root
[cpandey@cpandey #210]$ ls -ltr
total 34836
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 23:09 db-1.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 23:12 db-2.log
-rw-rw-r--. 1 cpandey cpandey    32824 Nov 18 23:12 db.redo
-rw-rw-r--. 1 cpandey cpandey   217088 Nov 18 23:12 db.data
[cpandey@cpandey #210]$ 
JBossFuse:karaf@root activemq:producer --user admin --password admin --messageCount 35 --messageSize 1000000 --destination BAR
JBossFuse:karaf@root activemq:dstat queues
Name                                                Queue Size  Producer #  Consumer #   Enqueue #   Dequeue #   Forward #    Memory %
A.B.C                                                        0           0           0           0           0           0           0
A.N.C                                                        0           0           0           0           0           0           0
BAR                                                         35           0           0          35           0           0          33
FOO                                                         35           0           0          35           0           0          33
Q.1                                                          0           0           0           0           0           0           0
Q.1.2                                                        0           0           0           0           0           0           0
Q.1.T                                                        0           0           0           0           0           0           0
Q.T                                                          0           0           0           0           0           0           0
Test.1                                                       0           0           0           0           0           0           0
Test.2                                                       0           0           0           0           0           0           0
Test.2.3                                                     0           0           0           0           0           0           0
X.Y.Z                                                        0           0           0           0           0           0           0
JBossFuse:karaf@root
[cpandey@cpandey #210]$ ls -ltr
total 69044
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 23:09 db-1.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 23:14 db-2.log
-rw-rw-r--. 1 cpandey cpandey 33554432 Nov 18 23:14 db-3.log
-rw-rw-r--. 1 cpandey cpandey    32824 Nov 18 23:14 db.redo
-rw-rw-r--. 1 cpandey cpandey   217088 Nov 18 23:14 db.data
[cpandey@cpandey #210]$
```

我希望本文能让您更好地理解 mKahaDB 及其实现和好处。

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

*Last updated: November 27, 2017*
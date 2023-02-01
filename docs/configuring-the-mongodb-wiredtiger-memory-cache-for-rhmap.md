# 为 RHMAP 配置 MongoDB WiredTiger 内存缓存

> 原文：<https://developers.redhat.com/blog/2018/09/17/configuring-the-mongodb-wiredtiger-memory-cache-for-rhmap>

本文描述了如何在[红帽移动应用平台](https://developers.redhat.com/products/mobileplatform/overview/) (RHMAP)中配置 MongoDB 的 WiredTiger 内存缓存，以防止高使用率内存问题和 Nagios 警报。如果 WiredTiger 缓存耗尽了容器的所有可用内存，就会出现内存问题和 Nagios 警报。

WiredTiger 存储引擎是 MongoDB 版中的默认存储引擎。它使用多版本并发控制( [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) )架构进行写操作，以便允许同时对同一文档进行多个不同的修改。

WiredTiger 还缓存数据并创建检查点，以便您能够在必要时随时进行恢复。例如，如果部署在容器中的 MongoDB 映像失败，那么恢复没有持久化的数据是很有用的。此外，WiredTiger 可以使用其日志文件恢复未设置检查点的数据。有关更多信息，请参见[日志文档](https://docs.mongodb.com/manual/core/wiredtiger/#journal)和[快照和检查点文档](https://docs.mongodb.com/manual/core/wiredtiger/#storage-wiredtiger-checkpoints)。

## 配置内存使用

为了防止内存问题和 Nagios 警报，属性`storage.wiredTiger.engineConfig.cacheSizeGB`应该设置为小于容器中可用 RAM 数量的值。实现相同结果的另一个选项是通过`–wiredTigerCacheSizeGB`参数。

在 MongoDB 3.2 版本中，当可用内存量大于 1GB 时，默认配置是使用 1GB 内存或 60%的可用内存量。在 MongoDB 的 3.4 版本中，这个使用百分比被替换为 50%。更多细节请参见[storage . wired tiger . engine config . cachesizegb API 文档](https://docs.mongodb.com/manual/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB)。

由于 MongoDB 3.2 中的一个错误，需要一个变通方法来接受低于 1Gi 的值。属性`configString: cache_size=<cache-size>`需要在 MongoDB 配置中设置。以下是将属性设置为 600MB 的示例:

```
# storage options - How and where to store data
storage:
    # Directory for datafiles (defaults to /data/db/)
    dbPath: ${MONGODB_DATADIR}
    wiredTiger:
       engineConfig:
           configString: cache_size=600M
```

**注:**bug*“wired tiger 缓存大小只能以整千兆字节为单位配置”*此处可查看。

## 在 RHMAP 中避免高使用率内存警报和与 MongoDB 相关的问题

在 RHMAP 4 . 6 . 5 或更高版本中，WiredTiger 内存缓存已经配置为使用 60%/600MB 的默认可用内存量，在发布的较新 RHMAP MongoDB 映像中是 1GB。要查看这张图片，请看它的目录[这里](https://access.redhat.com/containers/#/registry.access.redhat.com/rhmap46/mongodb)。但是，在前面的映像中，Nagios 中可能会触发内存问题和高使用率内存警报，因为没有设置缓存大小配置。

为了获得最佳用户体验并避免技术欠账，建议使用产品的最新版本。如果这是不可能的，那么建议更新所有 MongoDB 实例的配置，以便在两个项目(Core 和 MBaaS)中包含 WiredTiger 缓存大小的正确值。

以下步骤是 RHMAP 高级软件工程师 Shannon Poole 建议手动执行此配置的示例。

1.创建一个[配置映射](https://docs.openshift.com/container-platform/3.9/dev_guide/configmaps.html)，如下例所示:

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: wired-tiger-config
  namespace: rhmap-3-node-mbaas
data:
  mongod.conf: |
    ##
    ## For list of options visit:
    ## https://docs.mongodb.org/manual/reference/configuration-options/
    ##

    # systemLog options - How to do logging
    systemLog:
      # Runs the mongod in a quiet mode that attempts to limit the 
      # amount of output
      quiet: true

    # net options - Network interfaces settings
    net:
      # Specify port number (27017 by default)
      port: 27017

    # storage options - How and where to store data
    storage:
      # Directory for datafiles (defaults to /data/db/)
      dbPath: /var/lib/mongodb/data
      wiredTiger:
        engineConfig:
          configString: cache_size=400M

    # replication options - Configures replication
    replication:
      # Specifies a maximum size in megabytes for the replication 
      # operation log (i.e. the oplog,
      # 5% of disk space by default)
      oplogSizeMB: 64

```

2.将[配置图](https://docs.openshift.com/container-platform/3.9/dev_guide/configmaps.html)添加到 MongoDB pod 的[部署配置(dc)](https://docs.openshift.com/container-platform/3.9/dev_guide/deployments/basic_deployment_operations.html) 中，以便将其安装在正确的位置(`/etc/mongod.conf`)。要编辑部署配置，请使用以下命令或 Red hat OpenShift 控制台。

```
$ oc edit dc/<deployment_config>
```

**注:**参见 Red Hat OpenShift 文档的[批量消耗](https://docs.openshift.com/container-platform/3.9/dev_guide/configmaps.html#configmaps-use-case-consuming-in-volumes)部分了解更多信息。

3.更新 MongoDB 副本集容器规范，如下例所示:

```
containers:
- volumeMounts:
  - name: wired-tiger-config
    mountPath: /etc/mongod.conf
    subPath: mongod.conf
volumes:
- name: wired-tiger-config
  configMap:
    name: wired-tiger-config
```

4.检查 pod 日志以验证更改。看看在 MongoDB 重新部署后，自定义缓存大小是否还会存在。当 MongoDB 初始化时，将在日志中描述这种定制，如下例所示:

```
=> [Mon Aug 20 13:42:22] wiredTiger cacheSizeGB set to 1
=> [Mon Aug 20 13:42:22] Waiting for local MongoDB to accept connections  …
2018-08-20T13:42:22.837+0000 I STORAGE  [main] Engine custom option: cache_size=600M
```

## 结论

如果为 MongoDB 实例分配的内存量将发生变化，那么建议您检查配置并为 WiredTiger 缓存设置一个合适的值。当内存资源大于 1GB 时，没有必要执行此配置，因为在这种情况下，默认设置是总可用资源的 60%。但是，如果为了分配 1GB 或更少的空间而进行了更改，建议自定义该值，如本文所述。

*Last updated: September 14, 2018*
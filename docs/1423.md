# 第 1 部分:集装箱式储存

> 原文：<https://developers.redhat.com/blog/2017/08/21/gluster-for-openshift-part-1-container-ready-storage>

OpenShift 容器平台(OCP)提供了许多不同类型的持久存储。持久存储确保数据在构建和容器迁移之间应该是持久的。当选择持久存储后端以确保后端支持项目所需的扩展、速度、动态配置、RWX/RWO 支持和冗余时。容器就绪存储(CRS)或 OCP 的本机 Gluster 是由持久卷的概念定义的，持久卷是 OCP 创建的对象，允许定义存储，然后由 pod 使用以实现数据持久性。

持久卷(PV)的请求是通过使用持久卷声明(PVC)来完成的。当该声明被系统成功实现时，它还将把持久存储安装到一个或多个 pod 内的特定目录。这个目录被称为挂载路径，并使用一个称为绑定挂载的概念来简化。

OpenShift ansible contrib repo 为许多平台提供商提供参考架构，包括 AWS、Azure、GCE、OpenStack、RHEV 和 VMware。

github repo 以及用于在 VMware 上部署 OpenShift 和 CRS 的行动手册和脚本位于以下位置:

[https://github . com/open shift/open shift-ansi ble-contrib/tree/master/reference-architecture/VMware-ansi ble](https://github.com/openshift/openshift-ansible-contrib/tree/master/reference-architecture/vmware-ansible)。

这些行动手册和脚本将全程指导您利用容器就绪型存储在 VMware vCenter 上部署 OCP。

### 部署容器就绪存储

openshift-ansible-contrib git 存储库中提供了一个名为 add-node.py 的 python 脚本。当 add-node.py 与- node_type=storage 选项一起使用时，以下操作将完全自动化(取决于 ocp-on-vmware.ini 文件中的变量“container_storage=cns”)。

1.  创建三个具有 32 GB 内存和 2 个虚拟 CPU 的 VMware 虚拟机。
2.  向 Red Hat 注册新机器。
3.  在每台机器上安装 g luster CRS 的必备组件。
4.  向每个节点添加一个 VMDK 卷，作为用于 CRS 的可用块设备。
5.  使用虚拟机主机名和新的 VMDK 设备名创建一个 heketi topology.json 文件。
6.  在其中一个 CRS 节点上安装 heketi 和 heketi-cli 软件包。
7.  将 heketi 公钥复制到所有 CRS 节点。
8.  使用用户提供的管理员和用户密码以及其他必要配置修改 heketi.json 文件，以便对所有 CRS 节点进行无密码 SSH。
9.  使用 heketi-cli 和 topology.json 文件部署新的 CRS 集群。
10.  为 PVC 创建创建 heketi-secret 和新的 StorageClass 对象。

下面是上面第 9 步自动化的一个例子。加载 CRS topology.json 文件以创建新的 CRS 可信存储池(TSP)。这是从部署了 heketi 的 CRS 节点完成的。topology.json 文件在此节点上存档，以便将来修改以添加更多存储设备、更多存储节点等。

```
$ cat topology.json
{
    "clusters": [
        {
            "nodes": [
                {
                    "devices": [
                        "/dev/sdd"
                    ],
                    "node": {
                        "hostnames": {
                            "manage": [
                                "ocp3-crs0.dpl.local"
                            ],
                            "storage": [
                                "172.0.10.215"
                            ]
                        },
                        "zone": 1
                    }
                },
                {
                    "devices": [
                        "/dev/sdd"
                    ],
                    "node": {
                        "hostnames": {
                            "manage": [
                                "ocp3-crs1.dpl.local"
                            ],
                            "storage": [
                                "172.0.10.216"
                            ]
                        },
                        "zone": 2
                    }
                },
                {
                    "devices": [
                        "/dev/sdd"
                    ],
                    "node": {
                        "hostnames": {
                            "manage": [
                                "ocp3-crs2.dpl.local"
                            ],
                            "storage": [
                                "172.0.10.217"
                            ]
                        },
                        "zone": 3
                    }
                }
            ]
        }
    ]
}
```

现在导出 heketi 环境值并加载 topology.json 文件:

```
$ export HEKETI_CLI_SERVER=http://ocp3-crs-0.dpl.local:8080
$ export HEKETI_CLI_USER=admin
$ export HEKETI_CLI_KEY=myS3cr3tpassw0rd
$ heketi-cli topology load --json=topology.json
Creating cluster ... ID: bb802020a9c2c5df45f42075412c8c05
	Creating node ocp3-crs-0.dpl.local ... ID: b45d38a349218b8a0bab7123e004264b
				Adding device /dev/sdd ... OK
	Creating node ocp3-crs-1.dpl.local ... ID: 2b3b30efdbc3855a115d7eb8fdc800fe
				Adding device /dev/sdd ... OK
	Creating node ocp3-crs-2.dpl.local ... ID: c7d366ae7bd61f4b69de7143873e8999
				Adding device /dev/sdd ... OK
```

### 创建 Heketi secret 和 CRS 存储类 OCP 对象

对于上面的步骤 10，OCP 允许使用秘密，因此项目不需要以明文存储。在配置 heketi.json 文件期间指定的 heketi 的管理员密码应该存储在 base64 编码中。OCP 可以引用这个秘密，而不是以明文形式指定密码。

```
$ echo -n myS3cr3tpassw0rd | base64
bXlTM2NyM3RwYXNzdzByZA==
```

在安装了具有集群管理权限的 OCP 客户端的主服务器或工作站上，使用以下 YAML 中的 base64 密码字符串来定义 OCP 的默认名称空间中的密码。

```
$ cat heketi-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: heketi-secret
  namespace: default
data:
  key: bXlTM2NyM3RwYXNzdzByZA==
type: kubernetes.io/glusterfs
```

使用以下 OCP CLI 命令创建密码。

```
$ oc create -f heketi-secret.yaml
secret "heketi-secret" created
```

StorageClass 对象需要定义某些参数才能成功创建资源。使用前面步骤中导出的环境变量的值来定义 resturl、restuser、secretNamespace 和 secretName。使用 StorageClass 对象的主要好处是，可以使用读写一次(RWO)、只读一次(ROX)或读写多次(RWX)访问模式创建持久性存储。

```
$ cat storageclass.yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: crs-gluster
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://ocp3-crs-0.dpl.local:8080"
  restauthenabled: "true"
  restuser: "admin"
  secretNamespace: "default"
  secretName: "heketi-secret"
```

创建 StorageClass yaml 文件后，使用 oc create 命令在 OpenShift 中创建对象。

```
$ oc create -f storageclass.yaml
```

要验证是否创建了 StorageClass 对象，请执行以下操作:

```
$ oc get storageclass
NAME             TYPE
crs-gluster kubernetes.io/glusterfs

$ oc describe storageclass crs-gluster
Name:		crs-gluster
IsDefaultClass:	No
Annotations:	&amp;amp;lt;none&amp;amp;gt;
Provisioner:	kubernetes.io/glusterfs
Parameters:	restauthenabled=true,resturl=http://ocp3-crs-0.dpl.local:8080,restuser=admin,secretName=heketi-secret,secretNamespace=default
No events.
```

### 创建动态持久卷声明(PVC)

上一节中创建的存储类允许使用 CRS 资源动态调配存储。以下示例显示了从 crs-gluster StorageClass 对象请求的动态预配 gluster 卷。下面提供了一个永久卷声明示例:

```
$ vi db-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: db
 annotations:
   volume.beta.kubernetes.io/storage-class: crs-gluster
spec:
 accessModes:
  - ReadWriteOnce
 resources:
   requests:
     storage: 10Gi

$ oc create -f db-claim.yaml
persistentvolumeclaim "db" created
```

用所需的 StorageClass 对象名配置 OpenShift 模板也可以进行动态 PV 声明。下面的例子展示了如何修改默认的 openshift mysql-persistent 模板文件。如果未指定 StorageClass 对象名称，将使用默认的 storageclass(如果使用的话)。此外，PVC 的默认大小是 1GB，因此如果需要更大的大小，请确保增加该大小。

```
$ oc export template/mysql-persistent -n openshift -o yaml &amp;amp;gt; mysql-persistent.yaml
$ cat mysql-persistent.yaml
....omitted....
- apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: ${DATABASE_SERVICE_NAME}
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: ${VOLUME_CAPACITY}
....omitted....
```

使用所需的 StorageClass 对象名称修改模板，为 mount-path=/var/lib/mysql/data 创建动态 PVC 或 gluster 卷。

```
$ vim mysql-persistent.yaml
....omitted....
- apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: ${DATABASE_SERVICE_NAME}
    annotations:
      volume.beta.kubernetes.io/storage-class: crs-gluster
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: ${VOLUME_CAPACITY}
....omitted....
```

***...Gluster for OpenShift - Part 2:容器-原生存储*** 即将推出！

* * *

### [在 VMWARE VCENTER 6 上部署 RED HAT open shift CONTAINER PLATFORM 3，利用 GLUSTER CONTAINER-NATIVE STORAGE](https://access.redhat.com/articles/3130131)**本参考架构介绍了如何在 VMWARE VCENTER 上部署和管理 RED HAT open shift CONTAINER PLATFORM，利用 g luster 实现持久存储。**

*Last updated: August 17, 2017*
# 1 PersistentVolume & PersistentVolumeClaim

Volume 提供了非常好的数据持久化方案，不过在可管理性上还有不足。

例如之前的AWS EBS，要使用Volume，Pod 必须事先知道如下信息：

1. 当前Volume 来自 AWS EBS
2. EBS Volume 已经提前创建，并且知道确切的volume-id。

Pod通常是由应用的开发人员维护，而Volume则通常是由存储系统的管理员维护。开发人员要获得上面的信息，要么询问管理员，要么自己就是管理员。

这样就带了一个管理上的问题：应用开发人员和系统管理员的职责耦合在一起了。

Kubernetes 给出的解决方案是 PersistentVolume 和 PersistentVolumeClaim。

PV（PersistentVolume）是外部存储系统中的一块存储空间，由管理员创建和维护。与Volume一样，PV 具有持久性，生命周期独立于Pod。

PVC (PersistentVolumeClaim) 是对PV的申请（Claim）。PVC 通常由普通用户创建和维护。需要为Pod分配存储资源时，用户可以创建一个PVC，指明存储资源的容量大小和访问模式（比如只读）等信息，Kubernetes 会查找并提供满足条件的PV。

有了PVC，用户只需要告诉Kubernetes 需要什么样的存储资源，而不需关心真正的空间从哪里分配、如何访问等底层细节信息。这些Storage Provider的底层信息交给管理员来处理，只有管理员才应该关心创建PV 的细节信息。

Kubernetes 支持多种类型的PV，比如AWS EBS、Ceph、NFS 等，完整列表参考：https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes

## 1.1 NFS PV

同样，我们通过NFS的示例来了解 PV 的使用方法。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /tmp
    server: 10.138.0.34
```

1. capacity 指定PV 的容量为1GB。
2. accessModes 指定访问模式为ReadWriteOnce
3. persistentVolumeReclaimPolicy 指定PV的回收策略为 Recycle
4. storageClassName 指定PV 的class为nfs。相当于为PV 设置了一个分类，PVC 可以指定class 申请相应 class 的 PV。
5. path 指定 PV在NFS 服务器上对应的目录。

### 1.1.1 accessModes

访问模式包括：
* ReadWriteOnce——该卷可以被单个节点以读/写模式挂载
* ReadOnlyMany——该卷可以被多个节点以只读模式挂载
* ReadWriteMany——该卷可以被多个节点以读/写模式挂载

在命令行中，访问模式缩写为：

* RWO - ReadWriteOnce
* ROX - ReadOnlyMany
* RWX - ReadWriteMany
一个卷一次只能使用一种访问模式挂载，即使它支持很多访问模式。

### 1.1.2 Reclaim Policy

回收策略包括：

* Retain（保留）—— 手动回收
* Recycle（回收）—— 清除PV中的数据（相当于 rm -rf /thevolume/*）
* Delete（删除）—— 删除Storage Provider上的对应存储资源（例如 AWS EBS、GCE PD、Azure Disk 和 OpenStack Cinder 卷）

当前，只有 NFS 和 HostPath 支持Recycle策略。AWS EBS、GCE PD、Azure Disk 和 Cinder 卷支持Delete策略。

### 1.1.3 storage Class

PV可以指定一个StorageClass来动态绑定PV和PVC，其中通过 storageClassName 属性来指定具体的StorageClass，如果没有指定该属性的PV，它只能绑定到不需要特定类的 PVC。

## 1.2 PVC

PersistentVolumeClaim（PVC）是用户存储的请求，PVC消耗PV的资源，可以请求特定的大小和访问模式，需要指定归属于某个Namespace，在同一个Namespace的Pod才可以指定对应的PVC。

当需要不同性质的PV来满足存储需求时，可以使用StorageClass来实现。

创建PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs
```

PVC只需要指定PV的容量、访问模式和class即可。

只有与 PVC 具有相同 storageClassName 的 PV 才能绑定到 PVC。

PVC可以不指定storageClassName，或者将该值设置为空，如果打开了准入控制插件，并且指定一个默认的 StorageClass，则PVC会使用默认的StorageClass，否则就绑定到没有StorageClass的 PV上。

## 1.3 使用PVC 作为 Volumn

将PVC作为Pod的Volume，PVC与Pod需要在同一个命名空间下，Pod的声明如下：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod1
spec:
  containers:
    - name: mypod1
      image: busybox
      args:
      - /bin/sh
      - -c
      - sleep 30000
      volumeMounts:
      - mountPath: "/mydata"
        name: mydata
  volumes:
    - name: mydata
      persistentVolumeClaim:    # 使用PVC
        claimName: mypvc1
```

## 1.4 回收PV

当不需要使用PV 时，可用删除PVC回收 PV

```bash
$ kubectl delete pvc mypvc1

$ kubectl get pod -o wide 

$ kubectl get pv
```

当PVC mypvc1 被删除后，会发现Kubernetes 启动了一个新的 Pod recycler-for-pv1，这个Pod 的作用就是清除PV pv1 的数据。此时 pv1 的状态为Released，表示已经解除了与 mypvc1 的绑定，正在清除数据，不过此时还不可用。

当数据清除完毕，pv1 的状态重新变为Available，此时可以被新的PVC申请。

因为PV的回收策略设置为Recycle，所以数据会被清除。如果我们希望保留数据，可以将策略设置为 Retain。

如果需要其能被其他PVC 申请，可以删除并重新创建pv1，删除操作只是删除了PV 对象，存储空间中的数据并不会被删除。

PV 还支持Delete 的回收策略，会删除PV 在Storage Provider 上对应的存储空间。NFS 的 PV 还不支持Delete，支持Delete 的 Provider有AWS EBS、GCE PD、Azure Disk 和 OpenStack Cinder Volume 等

## 1.5 PV 动态供给

前面讲述的提前创建PV，然后通过PVC 申请PV 并在 Pod 中使用，这种方式叫做静态供给（Static Provision）。

与之对应的是动态供给（Dynamical Provision），即如果没有满足PVC条件的PV，会动态创建PV。相比静态供给，动态供给有明显的优势：不需要提前创建PV，减少了管理员的工作量，效率高。

动态供给是通过 StorageClass 实现的，StorageClass 定义了如何创建PV。详见[StorageClass](StorageClass.md)

### 1.5.1 开启Dynamic Provisioning

需要先提前创建StorageClass对象，StorageClass中定义了使用哪个provisioner，并且在provisioner被调用时传入哪些参数，详见[StorageClass](StorageClass.md)。

* 磁盘类存储

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: slow
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-standard
    ```

* SSD类存储

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
    ```

### 1.5.2 使用 Dynamic Provisioning

创建一个PVC对象，并且在其中storageClassName字段指明需要用到的StorageClass的名称，例如：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

当使用到PVC的时候会自动创建对应的外部存储，当PVC被删除的时候，会自动销毁（或备份）外部存储。

### 1.5.3 默认的StorageClass

当没有对应的StorageClass配置时，可以设定默认的StorageClass，需要执行以下操作：

1. 在API Server开启 [DefaultStorageClass admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass) 。
2. 设置默认的StorageClass对象。

可以通过添加`storageclass.kubernetes.io/is-default-class`注解的方式设置某个`StorageClass`为默认的`StorageClass`。当用户创建了一个`PersistentVolumeClaim`，但没有指定`storageClassName`的时候，会自动将该PVC的`storageClassName`指向默认的`StorageClass`。

## 参考

* [Provisioner - Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner)
* CloudMan. 每天 5 分钟玩转 Kubernetes[M]. 清华大学出版社, 2018.

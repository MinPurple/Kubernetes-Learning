## Kubernetes是什么
* k8s是Google开源的容器集群管理系统
* 用于容器化应用程序的部署、扩展和管理
* k8s 目标是管理跨多个主机的容器，让部署容器化应用简单高效
* K8S在Docker容器技术的基础之上，大大地提高了容器化部署应用简单高效。并且具备了完整的集群管理能力，例如服务发现、资源调度、部署管理、缩容扩容、动态更新、持久化存储、监控、日志等，涵盖项目周期的各个环节。

## Docker 与Kubernetes 的联系
说到这里，就涉及到容器云平台核心组成了。

Docker是一个容器引擎，用于运行容器，Kubernetes是一个容器编排系统，不具备容器引擎功能，相比Docker是一个更高级封装，而他们在一起堪称珠联璧合，一起搞大事！如图：
![enter description here](/images/Kubernetes与Docker的联系.png)


## Kubernetes核心概念
### Kubernetes架构及相关组件
#### Kubernetes 架构
![enter description here](/images/Kubernetes架构.png)
**Master** 负责管理集群, master 协调集群中的所有活动，例如调度应用程序、维护应用程序的所需状态、扩展应用程序和滚动更新。

**Node节点** 是 Kubernetes 集群中的工作机器，可以是物理机或虚拟机。每个工作节点都有一个 kubelet，它是管理节点并与 Kubernetes Master 节点进行通信的代理。节点上还应具有处理容器操作的容器运行时，例如 Docker 或 rkt。一个 Kubernetes 工作集群至少有三个节点。 

Master 管理集群，而 节点 用于托管正在运行的应用程序。

#### Kubernetes系统组件
![enter description here](/images/Kubernetes系统组件.png)
- **Master组件：**
  - **API Server** ：
    
    提供了资源操作的唯一入口，集群的统一入口，各组件协调者，以RESTful API 提供接口服务，所有对象资源的增删改查和监听操作都交给APIServer处理后再提交给Etcd存储;

    Kubernetes 中所有的组件都会和 API Server 进行连接，组件与组件之间一般不进行独立的连接，都依赖于 API Server 进行消息的传送；
  - **etcd** :
    是一个分布式的一个key-value存储系统，保存整个集群的状态；
  - **controller manager**:
    负责维护集群的状态，处理集群中常规后台任务，不同的controller管理不同的资源，而Controller Manager就是负责管理这些控制器的；
  - **scheduler**
    负责资源的调度，按照预定的调度策略将实例(Pod)调度到相应的主机上；
  - **Pod网络**
    Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络，例如flannel。

- **Node组件：**
  - **kubelet**：
    kubelet是Master在Node节点上的Agent，管理本机运行容器的生命周期，比如创建容器、Pod挂载数据卷、下载secret、获取容器和节点状态等工作。kubelet将每个Pod转换成一组容器。

  - **kube-proxy**：

    负责为应用提供集群内部的服务发现和负载均衡

    service 在逻辑上代表了后端的多个Pod，外界通过service访问Pod。service接收到的请求时如何转发到Pod的呢？这就是kube-proxy需要完成的工作。
  
    每个Node都会运行kube-proxy服务，他负责将访问service 的TCP/UDP 数据流转发到后端的容器，如果有多个副本，kube-proxy会实现负载均衡。
  - **container runtime**：
    负责镜像管理以及容器的真正执行(CRI)。

### 基本对象概念
![enter description here](/images/Kubernetes概览.png)

#### Pod
* 是一组相关联的容器集合，由一个或多个容器组成
* 是Kubernetes调度的最小部署单元
* Pod中容器 共享 同一个网络命名空间，IP地址和端口空间
* Pod是短暂的而不是长久的应用。Pods被调度到节点，保持在这个节点上直到被销毁。

![enter description here](/images/Kubernetes系统组件.png)

#### Label
* 是识别Kubernetes对象的标签，以key/value 的方式附加到对象上，用于区分对象
* Label 不提供唯一性，并且实际上经常是很多对象(如Pods)都使用相同的 label 来标志具体的应用
* 使用 Label Selector 来选择一组相同 label 的对象
* Label Selector支持以下几种方式：
  * 等式，如：app=nginx
  * 集合，如env in (production, qa)
  * 多个label(AND关系)，如app=nginx,env=test

#### Service
* 是应用服务的抽象，定义了Pod逻辑集合和访问这个Pod集合的策略，定义了服务的访问入口
* Service代理Pod集合对外表现是为一个访问入口，分配一个集群IP地址，来自这个IP的请求将负载均衡转发后端Pod中的容器。
* Service通过LableSelector选择一组Pod提供服务。
* 支持ClusterIP， NodePort以及LoadBalancer三种类型
* Service的底层实现有userspace、 iptables和ipvs三种模式

![enter description here](/images/Service.png)

#### Volume
* 数据卷，共享Pod中容器使用的数据

#### Namespace
* 命名空间，将对象逻辑上隔离
* Namespace 是对一组资源和对象的抽象集合，将对象逻辑上分配到不同的Namespace，比如可以用来将系统内部的对象划分为不同的项目组或用户组，并设定控制策略，从而实现多租户。
* 命名空间也称为虚拟集群。
* 常见的 pods, services,deployments 等都是属于某一个 namespace 的(默认是default)，而 Node, PersistentVolumes 等则不属于任何 namespace。
  
#### Controller
* Kubernetes通常不会直接创建Pod，而是通过Controller来管理Pod的，Controller中定义了Pod的部署特性，比如有几个副本、在什么样的Node上运行等。为了满足不同的业务场景，Kubernetes提供了多种Controller，包括Deployment、ReplicaSet、DaemonSet、StatefulS、Job等。

#### ReplicaSet
* 是 Pod 副本的抽象，用于解决 Pod 的扩容和伸缩，并提供声明式更新等功能。
* 下一代ReplicationController，RC与RS唯一区别就是lableselector支持不同，RS支持新的基于集合的标签，RC仅支持基于等式的标签。

#### Deployment
* 无状态工作负载
* 管理ReplicaSets和Pod
* Deployment集成了上线部署、滚动升级、创建副本、暂停上线任务、恢复上线任务、回滚到以前某一版本(成功/稳定)的Deployment等功能，在某种程度上，Deployment 可以帮助我们实现无人值守的上线，大大降低我们的上线过程的复杂沟通、操作风险；
* Deployment 确保任意时间都有指定数量的 Pod "副本"在运行。如果为某个 Pod 创建了Deployment 并且指定3个副本，它会创建3个 Pod，并且持续监控它们。如果某个 Pod 不响应，那么 Deployment 会替换它，保持总数为3;如果之前不响应的 Pod 恢复了，现在就有4个 Pod 了，那么 Deployment 会将其中一个终止保持总数为3;如果在运行中将副本总数改为5，Deployment 会立刻启动2个新 Pod，保证总数为5。Deployment 还支持回滚和滚动升级。
* 当创建 Deployment 时，需要指定两个东西：
  * Pod模板：用来创建 Pod 副本的模板
  * Label标签：Deployment 需要监控的 Pod 的标签。
* 如果已经创建了Pod的一些副本，则通过**Service**在这些副本上进行负载均衡
* Service 通过 Label Selector 同后端的 Pod 副本建立关系，Deployment 保证后端Pod 副本的数量，也就是保证服务的伸缩性。 
* 官方建议使用Deployment管理ReplicaSets，而不是直接使用ReplicaSets。

#### StatefulSet
* 有状态应用，用于解决各个pod实例独立生命周期管理，提供各个实例的启动顺序和唯一性
* StatefulSet 能够保证Pod的每个副本在整个生命周期中名称是不变的，而其他Controller不提供这个功能。
* StatefulSet适合持久性的应用程序，有唯一的网络标识符(IP)，持久存储，有序的部署、扩展、删除和滚动更新。
* eg：zookeeper集群，mysql主从

![enter description here](/images/StatefulSet.png)

#### DaemonSet
* DaemonSet能够让所有(或一些)节点运行同一个Pod。当节点加入Kubernetes集群中，Pod会被(DaemonSet)调度到该节点上运行，当节点从Kubernetes集群中移除时，被DaemonSet调度的Pod会被删除。删除DaemonSet会清理它所有相关的Pod。
* 在使用Kubernetes来运行应用时，很多时候我们需要在一个区域(zone)或者所有Node上运行同一个守护进程(pod)，例如以下场景：
  * 每个Node上运行一个分布式存储的守护进程，例如glusterd，ceph
  * 运行日志采集器在每个Node上，例如fluentd，logstash
  * 运行监控的采集端在每个Node，例如prometheus node exporter，collectd等

![enter description here](/images/DaemonSet.png)

#### Job
* 用于运行结束就删除的应用，而其他Controller中的Pod通常是长期持续运行的
* 一次性任务，运行完成后Pod销毁，不再重新启动新容器
* 分位普通任务(Job)和定时任务(CronJob)


### 组件通信 
Kubernetes 多组件之间的通信原理：
* apiserver 负责 etcd 存储的所有操作，且只有 apiserver 才直接操作 etcd 集群
* apiserver 对内（集群中的其他组件）和对外（用户）提供统一的 REST API，其他组件均通过 apiserver 进行通信
  * controller manager、scheduler、kube-proxy 和 kubelet 等均通过 apiserver watch API 监测资源变化情况，并对资源作相应的操作
  * 所有需要更新资源状态的操作均通过 apiserver 的 REST API 进行
* apiserver 也会直接调用 kubelet API（如 logs, exec, attach 等），默认不校验 kubelet 证书，但可以通过 --kubelet-certificate-authority 开启（而 GKE 通过 SSH 隧道保护它们之间的通信）

#### 创建Pod的流程
![enter description here](/images/创建Pod的流程.png)

1. 用户通过 REST API 创建一个 Pod
2. apiserver 将其写入 etcd
3. scheduluer 检测到未绑定 Node 的 Pod，开始调度并更新 Pod 的 Node 绑定
4. kubelet 检测到有新的 Pod 调度过来，通过 container runtime 运行该 Pod
5. kubelet 通过 container runtime 取到 Pod 状态，并更新到 apiserver 中


## 参考：
* [Kubernetes 组件](https://kubernetes.io/zh/docs/concepts/overview/components/)
* [Kubernetes 初体验](https://www.qikqiak.com/k8s-book/docs/14.Kubernetes%E5%88%9D%E4%BD%93%E9%AA%8C.html)
* [基本概念与组件](https://www.qikqiak.com/k8s-book/docs/15.%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E4%B8%8E%E7%BB%84%E4%BB%B6.html)
* [云原生技术的前世今生](https://bbs.huaweicloud.com/videos/428f0ca6bf9a4a9c9dd8dd6eb4ae5919)
* CloudMan. 每天 5 分钟玩转 Kubernetes[M]. 清华大学出版社, 2018.
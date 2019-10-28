## 1 查看系统Event事件

```sh
$ kubectl describe pod <PodName> --namespace=<NAMESPACE> 
```

该命令可以显示Pod创建时的配置定义、状态等信息和最近的Event事件，事件信息可用于排错。例如当Pod状态为Pending，可通过查看Event事件确认原因，一般原因有几种：

1. 没有可用的Node可调度
2. 开启了资源配额管理并且当前Pod的目标节点上恰好没有可用的资源
3. 正在下载镜像（镜像拉取耗时太久）或镜像下载失败。

> kubectl describe还可以查看其它k8s对象：NODE,RC,Service,Namespace,Secrets。

### 1.1 Pod

```sh
$ kubectl describe pod <PodName> --namespace=<NAMESPACE> 
```

### 1.2 NODE

```sh
$ kubectl describe node <Node-IP>
```
### 1.3 RC

```sh
$ kubectl describe rc <RC> --namespace=<NAMESPACE>
```

### 1.4 NAMESPACE

```sh
$ kubectl describe namespace <NAMESPACE>
```

### 1.5 Service

```sh
$ kubectl describe service <SERVICE> --namespace=<NAMESPACE>
```

## 2 查看容器日志

### 2.1 查看指定pod的日志

```sh
$ kubectl logs <pod_name>

$ kubectl logs -f <pod_name> # 类似tail -f的方式查看
```

### 2.2 查看上一个pod的日志

```sh
$ kubectl logs -p <pod_name>
```

### 2.3 查看指定pod中指定容器的日志

```sh
$ kubectl logs <pod_name> -c <container_name>
```

### 2.4 kubectl logs --help

```sh
$ kubectl logs --help  

Print the logs for a container in a pod. If the pod has only one container, the container name is optional.  
Usage:  
  kubectl logs [-f] [-p] POD [-c CONTAINER] [flags]  
Aliases:  
  logs, log  
   
Examples:  
# Return snapshot logs from pod nginx with only one container  
$ kubectl logs nginx  
# Return snapshot of previous terminated ruby container logs from pod web-1  
$ kubectl logs -p -c ruby web-1  
# Begin streaming the logs of the ruby container in pod web-1  
$ kubectl logs -f -c ruby web-1  
# Display only the most recent 20 lines of output in pod nginx  
$ kubectl logs --tail=20 nginx  
# Show all logs from pod nginx written in the last hour  
$ kubectl logs --since=1h nginx  
```

## 3 查看k8s服务日志

### 3.1 journalctl

在Linux系统上systemd系统来管理kubernetes服务，并且journal系统会接管服务程序的输出日志，可以通过`systemctl status` 或`journalctl -u -f`来查看kubernetes服务的日志。

其中kubernetes组件包括：

1. kube-apiserver
2. kube-controller-manager   Pod扩容相关或RC相关
3. kube-scheduler            Pod扩容相关或RC相关
4. kubelet                   Pod生命周期相关：创建、停止等
5. etcd

### 3.2 日志文件

也可以通过指定日志存放目录来保存和查看日志

* --logtostderr=false：不输出到stderr
* --log-dir=/var/log/kubernetes:日志的存放目录
* --alsologtostderr=false:设置为true表示日志输出到文件也输出到stderr
* --v=0:glog的日志级别
* --vmodule=gfs*=2,test*=4：glog基于模块的详细日志级别

## 4 常见问题

### 4.1 Pod状态一直为Pending

```sh
$ kubectl describe pod <PodName> --namespace=<NAMESPACE> 
```

查看该POD的事件:

* 正在下载镜像但拉取不下来（镜像拉取耗时太久）[一般都是该原因]
* 没有可用的Node可调度
* 开启了资源配额管理并且当前Pod的目标节点上恰好没有可用的资源

解决方法：

* 查看该POD所在宿主机与镜像仓库之间的网络是否有问题，可以手动拉取镜像
* 删除POD实例，让POD调度到别的宿主机上

### 4.2 Pod创建后不断重启

`kubectl get pods`中Pod状态一会`running`，一会不是，且`RESTARTS`次数不断增加。

一般原因为容器启动命令不是阻塞式命令，导致容器运行后马上退出。

非阻塞式命令：

* 本身CMD指定的命令就是非阻塞式命令
* 将服务启动方式设置为后台运行

解决方法：

1. 将命令改为阻塞式命令（前台运行），例如：`zkServer.sh start-foreground`

2. java运行程序的启动脚本将 `nohup xxx &` 的`nobup`和`&`去掉，例如：

```sh
nohup JAVA_HOME/bin/java JAVA_OPTS -cp $CLASSPATH com.cnc.open.processor.Main &
```

改为：

```sh
JAVA_HOME/bin/java JAVA_OPTS -cp $CLASSPATH com.cnc.open.processor.Main
```
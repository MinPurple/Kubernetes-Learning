## 1 Kubectl 命令

### 1.1 kubectl 命令语法

```sh
kubectl [command] [TYPE] [NAME] [flags]
```

其中command，TYPE，NAME，和flags分别是：

* command: 指定要在一个或多个资源进行操作，例如create，get，describe，delete。

* TYPE：指定资源类型。资源类型区分大小写，可以指定单数，复数或缩写形式。例如，以下命令产生相同的输出：

    ```sh
    $ kubectl get pod pod1  
    $ kubectl get pods pod1 
    $ kubectl get po pod1
    ```

* NAME：指定资源的名称。名称区分大小写。如果省略名称，则会显示所有资源的详细信息,比如`$ kubectl get pods`。

    按类型和名称指定多种资源：

    * 要分组资源，如果它们都是相同的类型：`TYPE1 name1 name2 name`
  
        例: `$ kubectl get pod example-pod1 example-pod2`

    * 要分别指定多种资源类型:  `TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name`
  
        例: `$ kubectl get pod/example-pod1 replicationcontroller/example-rc1`

* flags：指定可选标志。例如，您可以使用`-s`或`--serverflags`来指定Kubernetes API服务器的地址和端口。

## 2 更多命令介绍

```sh
$ kubectl

kubectl controls the Kubernetes cluster manager.

Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create         Create a resource from a file or from stdin.
  expose         Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run            Run a particular image on the cluster
  set            Set specific features on objects
  run-container  Run a particular image on the cluster. This command is deprecated, use "run" instead

Basic Commands (Intermediate):
  get            Display one or many resources
  explain        Documentation of resources
  edit           Edit a resource on the server
  delete         Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout        Manage the rollout of a resource
  rolling-update Perform a rolling update of the given ReplicationController
  scale          Set a new size for a Deployment, ReplicaSet, Replication Controller, or Job
  autoscale      Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate    Modify certificate resources.
  cluster-info   Display cluster info
  top            Display Resource (CPU/Memory/Storage) usage.
  cordon         Mark node as unschedulable
  uncordon       Mark node as schedulable
  drain          Drain node in preparation for maintenance
  taint          Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe       Show details of a specific resource or group of resources
  logs           Print the logs for a container in a pod
  attach         Attach to a running container
  exec           Execute a command in a container
  port-forward   Forward one or more local ports to a pod
  proxy          Run a proxy to the Kubernetes API server
  cp             Copy files and directories to and from containers.
  auth           Inspect authorization

Advanced Commands:
  apply          Apply a configuration to a resource by filename or stdin
  patch          Update field(s) of a resource using strategic merge patch
  replace        Replace a resource by filename or stdin
  convert        Convert config files between different API versions

Settings Commands:
  label          Update the labels on a resource
  annotate       Update the annotations on a resource
  completion     Output shell completion code for the specified shell (bash or zsh)

Other Commands:
  api-versions   Print the supported API versions on the server, in the form of "group/version"
  config         Modify kubeconfig files
  help           Help about any command
  plugin         Runs a command-line plugin
  version        Print the client and server version information

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

## 3 操作的常用资源对象

1. Node
2. Podes
3. Replication Controllers
4. Services
5. Namespace
6. Deployment
7. StatefulSet

### 3.1 具体对象类型及缩写:
```js
  * all
  * certificatesigningrequests (aka 'csr')
  * clusterrolebindings
  * clusterroles
  * componentstatuses (aka 'cs')
  * configmaps (aka 'cm')
  * controllerrevisions
  * cronjobs
  * customresourcedefinition (aka 'crd')
  * daemonsets (aka 'ds')
  * deployments (aka 'deploy')
  * endpoints (aka 'ep')
  * events (aka 'ev')
  * horizontalpodautoscalers (aka 'hpa')
  * ingresses (aka 'ing')
  * jobs
  * limitranges (aka 'limits')
  * namespaces (aka 'ns')
  * networkpolicies (aka 'netpol')
  * nodes (aka 'no')
  * persistentvolumeclaims (aka 'pvc')
  * persistentvolumes (aka 'pv')
  * poddisruptionbudgets (aka 'pdb')
  * podpreset
  * pods (aka 'po')
  * podsecuritypolicies (aka 'psp')
  * podtemplates
  * replicasets (aka 'rs')
  * replicationcontrollers (aka 'rc')
  * resourcequotas (aka 'quota')
  * rolebindings
  * roles
  * secrets
  * serviceaccounts (aka 'sa')
  * services (aka 'svc')
  * statefulsets (aka 'sts')
  * storageclasses (aka 'sc')
```

## 4 kubectl 命令分类

## 4.1 增

1. create:[Create a resource by filename or stdin]

2. run:[ Run a particular image on the cluster]

3. apply:[Apply a configuration to a resource by filename or stdin]

4. proxy:[Run a proxy to the Kubernetes API server ]

## 4.2 删

1. delete:[Delete resources ]

## 4.3 改

1. scale:[Set a new size for a Replication Controller]

2. exec:[Execute a command in a container]

3. attach:[Attach to a running container]

4. patch:[Update field(s) of a resource by stdin]

5. edit:[Edit a resource on the server]

6. label:[Update the labels on a resource]

7. annotate:[Auto-scale a replication controller]

8. replace:[Replace a resource by filename or stdin]

9. config:[config modifies kubeconfig files]

## 4.4 查

1. get:[Display one or many resources]

2. describe:[Show details of a specific resource or group of resources]

3. log:[Print the logs for a container in a pod]

4. cluster-info:[Display cluster info]

5. version:[Print the client and server version information]

6. api-versions:[Print the supported API versions]

## 5 Pod 相关命令

### 5.1 查询Pod

```sh
$ kubectl get pod -o wide --namespace=<NAMESPACE>
```

### 5.2 进入Pod

```sh
$ kubectl exec -it <PodName> /bin/bash --namespace=<NAMESPACE>

# 进入Pod中指定容器
kubectl exec -it <PodName> -c <ContainerName> /bin/bash --namespace=<NAMESPACE>
```

### 5.3 删除Pod

```sh
$ kubectl delete pod <PodName> --namespace=<NAMESPACE>

# 强制删除Pod，当Pod一直处于Terminating状态
kubectl delete pod <PodName> --namespace=<NAMESPACE> --force --grace-period=0

# 删除某个namespace下某个类型的所有对象
kubectl delete deploy --all --namespace=test
```

### 5.4 日志查看

```sh
# 查看运行容器日志 
$ kubectl logs <PodName> --namespace=<NAMESPACE>
# 查看上一个挂掉的容器日志 
$ kubectl logs <PodName> -p --namespace=<NAMESPACE> 
```

## 6 常用命令

### 6.1 Node隔离与恢复

> 说明：Node设置隔离之后，原先运行在该Node上的Pod不受影响，后续的Pod不会调度到被隔离的Node上。

1. Node 隔离

    ```sh
    # cordon命令
    $ kubectl cordon <NodeName>
    # 或者
    $ kubectl patch node <NodeName> -p '{"spec":{"unschedulable":true}}'
    ```

2. Node 恢复

    ```sh
    # uncordon
    $ kubectl uncordon <NodeName>
    # 或者
    $ kubectl patch node <NodeName> -p '{"spec":{"unschedulable":false}}'
    ```

### 6.2 kubectl label 

1. 固定Pod到指定机器

    ```sh
    $ kubectl label node <NodeName> namespace/<NAMESPACE>=true
    ```
2. 取消Pod固定机器

    ```sh
    $ kubectl label node <NodeName> namespace/<NAMESPACE>-
    ```

### 6.3 升级镜像

```sh
# 升级镜像
$ kubectl set image deployment/nginx nginx=nginx:1.15.12 -n nginx
# 查看滚动升级情况
$ kubectl rollout status deployment/nginx  -n nginx
```

### 6.4 调整资源值（扩缩容）

```sh
# 调整指定容器的资源值
$ kubectl set resources sts nginx-0 -c=agent --limits=memory=512Mi -n nginx
```

### 6.5 调整readiness probe

```sh
# 批量查看readiness probe timeoutSeconds
$ kubectl get statefulset -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.template.spec.containers[0].readinessProbe.timeoutSeconds}{"\n"}{end}'

# 调整readiness probe timeoutSeconds参数
$ kubectl patch statefulset nginx-sts --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/readinessProbe/timeoutSeconds", "value":5}]' -n nginx
```

### 6.6 调整tolerations属性

```sh
$ kubectl patch statefulset nginx-sts --patch '{"spec": {"template": {"spec": {"tolerations": [{"effect": "NoSchedule","key": "dedicated","operator": "Equal","value": "nginx"}]}}}}' -n nginx
```

### 6.7 查看所有节点的IP

```sh
$ kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[0].address}{"\n"}{end}'
```

### 6.8 查看当前k8s组件leader节点

当k8s集群高可用部署的时候，kube-controller-manager和kube-scheduler只能一个服务处于实际逻辑运行状态，通过参数--leader-elect=true来开启选举操作。以下提供查询leader节点的命令。

```sh
$ kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml

apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"lhm-cluster-test-m_05857aab-916e-491a-b2e1-efe6f27b45f4","leaseDurationSeconds":15,"acquireTime":"2019-10-26T01:51:39Z","renewTime":"2019-10-26T03:38:26Z","leaderTransitions":6}'
  creationTimestamp: "2019-10-17T03:27:04Z"
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "842168"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-controller-manager
  uid: 4f5b9b2f-c41c-42ab-aacc-cbd0453c25d8
```

### 6.9 修改副本数

```sh
$ kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```

### 6.10 批量删除pod

```sh
$ kubectl get po -n default |grep Evicted |awk '{print $1}' |xargs -I {} kubectl delete po  {} -n default
```

## 参考

* [Overview of kubectl - Kubernetes](https://kubernetes.io/docs/reference/kubectl/overview/)
* [Configure Access to Multiple Clusters - Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
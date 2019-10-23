## 1. Kubernetes 对象概述

Kubernetes对象是Kubernetes系统中的持久实体。Kubernetes使用这些实体来表示集群的状态。包括：

* 容器化应用正在运行(以及在哪些节点上)
* 这些应用可用的资源
* 关于这些应用如何运行的策略，如重启策略，升级和容错策略等

操作kubernetes对象（增删改查）需要通过[kubernetes API](https://kubernetes.io/docs/reference/)，一般有以下几种方式：

1. kubectl命令工具
2. Client library的方式，例如 [client-go](https://github.com/kubernetes/client-go)

## 2. Spec 与 Status

每个Kubernetes对象都包含两个嵌套对象字段，用于管理Object的配置：Object Spec和Object Status。

* Spec 由用户提供，描述用户期望的对象特征及集群状态。
* Status 由kubernetes集群提供和更新，描述kubernetes对象的实时状态。

任何时候，kubernetes都会控制集群的实时状态status与用户的预期状态spec一致。

例如：当你定义Deployment的描述文件，指定集群中运行3个实例，那么kubernetes会始终保持集群中运行3个实例，如果任何实例挂掉，kubernetes会自动重建新的实例来保持集群中始终运行用户预期的3个实例。

## 3. 对象描述文件

在Kubernetes中创建对象时，必须提供描述其所需Status的对象Spec，以及关于对象（如name）的一些基本信息。

当使用Kubernetes API创建对象（直接或通过kubectl）时，该API请求必须将该信息作为JSON包含在请求body中。通常，可以将信息提供给kubectl .yaml文件，在进行API请求时，kubectl将信息转换为JSON。

以下示例是一个.yaml文件，显示Kubernetes Deployment所需的字段和对象Spec：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

执行kubectl apply的命令来创建Deployment
```bash
$ kubectl apply -f nginx-deployment.yaml --record

deployment "nginx-deployment" created
```

## 4. 必须字段

* apiVersion：kubernetes API的版本
* kind：kubernetes对象的类型
* metadata：唯一标识该对象的元数据，包括name，UID，可选的namespace
* spec：标识对象的详细信息，不同对象的spec的格式不同，可以嵌套其他对象的字段。


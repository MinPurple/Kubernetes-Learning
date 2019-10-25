每个成功的软件平台都有一个优秀的打包系统，比如Debian、Ubuntu 的 apt，Red Hat、CentOS 的yum。Helm 则是Kubernetes 上的包管理器。

## Why Helm

Kubernetes能够很好地组织和编排容器，但它缺少一个更高层次的应用打包工具，而Helm就是解决这个问题的。

对于一个MySQL 服务，Kubernetes 需要部署下面这些对象：
1. Service，让外界能够访问到MySQL

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-mysql
      labels:
        app: my-mysql
    spec:
      selector:
        app: my-mysql
      ports:
        - name: mysql
        port: 3306
        targetPort: mysql
    ```
2. Secret，定义MySQL的密码

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: my-mysql
      labels:
        app: my-mysql
    type: Opaque
    data:
      mysql-username: YWRtaW4=
      mysql-password: MTIzNDU2
    ```

3. PersistentVolumeClaim，为MySQL 申请持久化存储空间

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-mysql
      labels:
        app: my-mysql
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
        storage: 8Gi
    ```

4. Deployment，部署MySQL Pod,并使用上面的这些支持对象

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-mysql
      labels:
        app: my-mysql
    spec:
      template:
        metadata:
          labels:
            app: my-mysql
        spec:
          containers:
          - name: my-mysql
            image: "mysql:5.7.14"
            env:
            - name: MYSQL_USERNAME
              valueFrom:
                secretKeyRef:
                  name: my-mysql
                  key: mysql-username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: my-mysql
                  key: mysql-password
            - name: MYSQL_DATABASE
              value: ""

            ports:
            - name: mysql
              containerPort: 3306
            volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
          volumes:
          - name: data
            persistentVolumeClaim:
              claimName: my-mysql 
    ```

我们可以将上面这些配置保存到对象各自的文件中，或者集中写进一个配置文件，然后通过 `kubectl apply -f` 部署。

如果应用只由一个或几个这样的服务组成，上面的部署方式完全足够了。

但是，如果我们开发的是微服务架构的应用，组成应用的服务可能多达十个甚至几十上百个，这种组织和管理应用的方式就不好使了。

Why Helm：

1. 很难管理、编辑和维护如此多的服务。每个服务都有若干配置，缺乏一个更高层次的工具将这些配置组织起来。
2. 不容易将这些服务作为一个整体统一发布。部署人员需要首先理解应用都包含哪些服务，然后按照逻辑顺序依次执行`kubectl apply`，即缺少一种工具来**定义应用与服务，以及服务与服务之间的依赖关系**。
3. 不能高效的共享和重用服务。比如两个应用都要用到MySQL服务，但配置的参数不一样，这两个应用只能分别复制一套标准的MySQL配置文件，修改后通过`kubectl apply` 部署。也就是说，**不支持参数化配置和多环境部署**。
4. 不支持应用级别的版本管理。虽然可以通过`kubectl rollout undo` 进行回滚，但这只能针对单个Deployment，**不支持整个应用的回滚**。
5. 不支持对部署的应用状态进行验证。比如是否能通过预定义的账号访问MySQL。虽然Kubernetes有健康检查，但那时针对单个容器，我们需要**应用（服务）级别的健康检查**。
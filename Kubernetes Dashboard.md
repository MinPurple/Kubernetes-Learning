## 1 搭建K8S Dashboard
### 1.1 安装

K8S Dashboard是官方的一个基于WEB的用户界面，专门用来管理K8S集群，并可展示集群的状态。K8S集群安装好后默认没有包含Dashboard，需要额外创建。

[官方文档](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)


```bash
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

将Service改成NodePort类型，注意 YAML 中最下面的 Service 部分新增一个type=NodePort
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard

```bash
$ kubectl create -f kubernetes-dashboard.yaml
```

Dashboard 会在kube-system namespace 中创建自己的Deployment 和Service：
```bash
$ kubectl --namespace=kube-system get deployment kubernetes-dashboard
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-dashboard   1/1     1            1           17h

$ kubectl --namespace=kube-system get service  kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.99.38.253   <none>        443:32352/TCP   17h
```

查看 dashboard 的外网访问端口:

`$ kubectl get svc kubernetes-dashboard -n kube-system`

查看 dashboard 运行在哪个node:

`$ kubectl get pods -A -o wide`

然后直接访问集群中的任何一个节点 IP 加上上面的端口即可打开 dashboard 页面了
https://34.83.12.235:32352/


### 1.2 配置登录权限

登录 Dashboard 的时候支持 Kubeconfig 和token 两种认证方式，Kubeconfig 中也依赖token 字段，所以生成token 这一步是必不可少的。

创建一个admin用户并授予admin 角色绑定，使用下面的yaml文件(dashboard-admin.yaml)创建admin用户并赋予他管理员权限，然后就可以通过token 登陆dashbaord，这种认证方式本质实际上是通过Service Account 的身份认证加上Bearer token请求 API server 的方式实现，参考 [Kubernetes 中的认证](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)。

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system

---
apiVersion: v1
kind: ServiceAccount
metadata:
name: admin
namespace: kube-system
labels:
kubernetes.io/cluster-service: "true"
addonmanager.kubernetes.io/mode: Reconcile
```
需要注意yaml文件的格式！

执行kubectl apply 使之生效：
```bash
$ kubectl apply -f dashboard-admin.yaml
```

上面的admin用户创建完成后我们就可以获取到该用户对应的token了，如下命令：
```bash
$ kubectl get secret -n kube-system|grep admin-token
admin-token-487vz                                kubernetes.io/service-account-token   3      97s
$ kubectl get secret admin-token-d5jsg -o jsonpath={.data.token} -n kube-system |base64 -d # 会生成一串很长的base64后的字符串
```
然后在 dashboard 登录页面上直接使用上面得到的 token 字符串即可登录，这样就可以拥有管理员权限操作整个 kubernetes 集群的对象，当然你也可以为你的登录用户新建一个指定操作权限的用户。​​ 
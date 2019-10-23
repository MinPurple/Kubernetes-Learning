## 1 Network Policy

网络策略(Network Policy)说明一组 Pod 之间是如何被允许互相通信，以及如何与其它网络 Endpoint 进行通信。 

Network Policy 是Kubernetes 的一种资源。Network Policy 通过Label选择Pod，并指定其他Pod或外界如何与这些Pod通信。

## 2 前提

网络策略通过网络插件来实现，所以用户必须使用支持 NetworkPolicy 的网络解决方案 - 简单地创建资源对象，而没有控制器来使它生效的话，是没有任何作用的。

## 3 隔离和非隔离的Pod

默认情况下，所有Pod是非隔离的，即任何来源的网络流量都能够访问Pod，没有任何限制。

Pod可以通过相关的网络策略进行隔离。当为Pod定义了Network Policy时，只有Policy允许的流量才能访问Pod。

不过，不是所有的Kubernetes网络方案都支持Network Policy。比如Flannel 就不支持，Calico是支持的。


## 4 Network Policy

下面是一个 NetworkPolicy 的示例:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

* 必填字段：与所有其他的Kubernetes配置一样，NetworkPolicy 需要 apiVersion、 kind和 metadata 字段
* spec: NetworkPolicy spec 中包含了在一个命名空间中定义特定网络策略所需的所有信息
* podSelector:  每个 NetworkPolicy 都包括一个 podSelector ，它对该策略所应用的一组Pod进行选择。因为 NetworkPolicy 目前只支持定义 ingress 规则，这里的 podSelector 本质上是为该策略定义 “目标pod” 。示例中的策略选择带有 “role=db” 标签的pod。空的 podSelector 选择命名空间下的所有pod。
* policyTypes： 每个 NetworkPolicy 都包括一个 policyTypes 列表，包括Ingress、Egress或者两者。policyTypes字段指示给定的策略是否适用于到选定Pod的入站流量、来自选定Pod的出站流量，或两者都适用。 如果在NetworkPolicy上未指定任何policyType，则默认情况下将始终设置Ingress，并且如果NetworkPolicy具有任何出口规则，则将设置Egress。
* ingress: 每个 NetworkPolicy 包含一个 ingress 规则的白名单列表。（其中的）规则允许同时匹配 from 和 ports 部分的流量。示例策略中包含一条简单的规则： 它匹配一个单一的端口，来自两个来源中的一个， 第一个通过 namespaceSelector 指定，第二个通过 podSelector 指定。
* egress: 每个 NetworkPolicy 包含一个 egress 规则的白名单列表。每个规则都允许匹配 to 和 port 部分的流量。该示例策略包含一条规则，该规则将单个端口上的流量匹配到 10.0.0.0/24 中的任何目的地。

所以，示例网络策略:
1. 隔离 “default” 命名空间下 “role=db” 的pod (如果它们不是已经被隔离的话)。
2. 允许从 “default” 命名空间下带有 “role=frontend” 标签的pod到 “default” 命名空间下的pod的6379 TCP端口的连接。
   * 标签为 “role=frontend” 的 “default” 命名空间中的任何Pod
   * 命名空间中带有标签 “project=myproject” 的任何pod
   * IP 地址范围为 172.17.0.0–172.17.0.255 和 172.17.2.0–172.17.255.255（即，除了 172.17.1.0/24 之外的所有 172.17.0.0/16）
3. 允许从带有 “project=myproject” 标签的命名空间下的任何 pod 到 “default” 命名空间下的 pod 的6379 TCP端口的连接。

## 5 默认策略

默认情况下，如果名称空间中不存在任何策略，则所有进出该名称空间中的Pod的流量都被允许。以下示例使你可以更改该名称空间中的默认行为。

### 5.1 默认拒绝所有入口流量

您可以通过创建选择所有容器但不允许任何进入这些容器的入口流量的 NetworkPolicy 来为名称空间创建 “default” 隔离策略。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

这样可以确保即使容器没有选择其他任何 NetworkPolicy，也仍然可以被隔离。此策略不会更改默认的出口隔离行为。

### 5.2 默认允许所有入口流量

如果要允许所有流量进入某个命名空间中的所有 Pod（即使添加了导致某些 Pod 被视为“隔离”的策略），则可以创建一个策略来明确允许该命名空间中的所有流量。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```

### 5.3 默认拒绝所有出口流量

可以通过创建选择所有容器但不允许来自这些容器的任何出口流量的 NetworkPolicy 来为名称空间创建 “default” egress 隔离策略。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

这样可以确保即使没有被其他任何 NetworkPolicy 选择的 Pod 也不会被允许流出流量。此策略不会更改默认的 ingress 隔离行为

### 5.4 默认允许所有出口流量

如果要允许来自命名空间中所有 Pod 的所有流量（即使添加了导致某些 Pod 被视为“隔离”的策略），则可以创建一个策略，该策略明确允许该命名空间中的所有出口流量。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  egress:
  - {}
  policyTypes:
  - Egress
```

### 5.5 默认拒绝所有入口和所有出口流量

可以为名称空间创建 “default” 策略，以通过在该名称空间中创建以下 NetworkPolicy 来阻止所有入站和出站流量。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

这样可以确保即使没有被其他任何 NetworkPolicy 选择的 Pod 也不会被允许进入或流出流量。

## 6 小结

Kubernetes 采用的是扁平化的网络模型，每个Pod 都有自己的IP，并且可以直接通信。

CNI 规范使得Kubernetes 可以灵活选择多种Plugin 实现集群网络。

Network Policy 赋予了Kubernetes 强大的网络访问控制机制。
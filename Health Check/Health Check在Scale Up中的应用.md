## Health Check在Scale Up中的应用

对于多副本应用，当执行`Scale Up` 操作时，新副本会作为`backend`被添加到`Service`的负载均衡中，与已有副本一起处理客户的请求。考虑到应用启动通常都需要一个准备阶段，比如加载缓存数据、连接数据库等，从容器启动到真正能够提供服务是需要一段时间的。我们可以通过`Readiness` 探测**判断容器是否就绪，避免将请求发送到还没有准备好的`backend`**。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myhttpd
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            scheme: HTTP
            path: /healthy
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  port:
  - protocol: TCP
    port: 8080
    targetPort: 80
```

重点关注 `readinessProbe` 部分，这里我们使用了不同于`exec`的另一种探测方法`httpGet`。Kubernetes对于该方法探测成功的判断条件是**http请求的返回代码在`200~400`之间**。

* scheme 指定协议，支持HTTP（默认值）和 HTTPS
* path 指定访问路径
* port 指定端口

上面配置的作用是：
1. 容器启动10秒之后开始探测
2. 如果`http://[container_ip]:8080/healthy` 返回代码不是 `200~400`，表示容器没有就绪，不接收 Service `web-svc` 的请求。
3. 每隔`5秒`探测一次
4. 直到返回代码为`200~400`，表名容器已经就绪，然后将其加入到`web-svc`的负载均衡中，开始处理客户请求。
5. 探测会继续以`5秒`的时间间隔执行，如果连续发生`3次`失败，容器又会从负载均衡中移除，直到下次探测成功重新加入。

对于`http://[container_ip]:8080/healthy`，应用则可以实现自己的判断逻辑，比如检查所依赖的数据库是否就绪等。

对于生产环境中重要的应用，都建议配置`Health Check`，**保证处理客户请求的容器都是准备就绪的`Service backend`**。
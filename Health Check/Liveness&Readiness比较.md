## Liveness 和 Readiness 比较

1. `Liveness` 探测和`Readiness`探测是两种 Health Check机制，如果不特意配置，Kubernetes将对两种探测采取相同的默认行为，即通过判断容器启动进程的返回值是否为零来判断探测是否成功。
2. 两种探测的配置方法完全一样，支持的配置参数也一样。不同之处在于探测失败后的行为：
   
   * `Liveness` 探测是重启容器
   * `Readiness` 探测则是将容器设置为不可用，不接收Service转发的请求。

3. `Liveness` 探测和 `Readiness` 探测是独立执行的，二者之间没有依赖，所以可以单独使用，也可以同时使用。

   * 用 `Liveness` 探测判断容器是否需要重启以实现自愈；
   * 用 `Readiness` 探测判断容器是否已经准备好对外提供服务。
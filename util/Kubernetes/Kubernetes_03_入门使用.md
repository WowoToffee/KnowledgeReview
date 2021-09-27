# Kubernetes 入门使用

## Pod

Pod，是 Kubernetes 项目中最小的 API 对象。Pod，是 Kubernetes 项目的原子调度单位。

### Pod 的实现原理

==Pod，其实是一组共享了某些资源的容器。== 具体的说：**Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。**


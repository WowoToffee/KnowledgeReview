# Kubernetes 入门使用

## Pod

Pod，是 Kubernetes 项目中最小的 API 对象。Pod，是 Kubernetes 项目的原子调度单位。

### Pod 的实现原理

==Pod，其实是一组共享了某些资源的容器。== 具体的说：**Pod 里的所有容器，共享的是同一个 Network Namespace，并且可以声明共享同一个 Volume。**

在 Kubernetes 中，有几种特殊的 Volume，它们存在的意义不是为了存放容器里的数据，也不是用来进行容器和宿主机之间的数据交换。这些特殊 Volume 的作用，是为容器提供预先定义好的数据。所以，从容器的角度来看，这些 Volume 里的信息就是仿佛是被Kubernetes“投射”（Project）进入容器当中的。这正是 Projected Volume 的含义。

到目前为止，Kubernetes 支持的 Projected Volume 一共有四种：

1. Secret；
2. ConfigMap；
3. Downward API；
4. ServiceAccountToken。

## 容器的健康检查和恢复机制

### 探针（Probe）

shareProcessNamespace: true



拉取镜像策略

 **Pod 对象在 Kubernetes 中的生命周期**。

Pod 生命周期的变化，主要体现在 Pod API 对象的**Status 部分**，这是它除了 Metadata 和 Spec 之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：

1. Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。
2. Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
3. Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
4. Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
5. Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

**Kubernetes 支持的 Projected Volume 一共有四种：**

1. Secret；
2. ConfigMap；
3. Downward API；
4. ServiceAccountToken。

Pod.yaml

## 控制器模式

### Deployment

### ReplicaSet

Deployment ----------------->ReplicaRet(v1)--------------------->Pod(3个)

备注：Deployment 控制 ReplicaSet（版本），ReplicaSet 控制 Pod（副本数）。这个两层控制关系一定要牢记。

Deployment 实际上是一个**两层控制器**。首先，它通过**ReplicaSet 的个数**来描述应用的版本；然后，它再通过**ReplicaSet 的属性**（比如 replicas 的值），来保证 Pod 的副本数量。



### StatefulSet

StatefulSet 的设计其实非常容易理解。它把真实世界里的应用状态，抽象为了两种情况：

1. **拓扑状态**。这种情况意味着，应用的多个实例之间不是完全对等的关系。这些应用实例，必须按照某些顺序启动，比如应用的主节点 A 要先于从节点 B 启动。而如果你把 A 和 B 两个 Pod 删除掉，它们再次被创建出来时也必须严格按照这个顺序才行。并且，新创建出来的 Pod，必须和原来 Pod 的网络标识一样，这样原先的访问者才能使用同样的方法，访问到这个新 Pod。
2. **存储状态**。这种情况意味着，应用的多个实例分别绑定了不同的存储数据。对于这些应用实例来说，Pod A 第一次读取到的数据，和隔了十分钟之后再次读取到的数据，应该是同一份，哪怕在此期间 Pod A 被重新创建过。这种情况最典型的例子，就是一个数据库应用的多个存储实例。

所以，**StatefulSet 的核心功能，就是通过某种方式记录这些状态，然后在 Pod 被重新创建时，能够为新 Pod 恢复这些状态。**

StatefulSet 的工作原理:

**首先，StatefulSet 的控制器直接管理的是 Pod**。这是因为，StatefulSet 里的不同 Pod 实例，不再像 ReplicaSet 中那样都是完全一样的，而是有了细微区别的。比如，每个 Pod 的 hostname、名字等都是不同的、携带了编号的。而 StatefulSet 区分这些实例的方式，就是通过在 Pod 的名字里加上事先约定好的编号。

**其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录**。只要 StatefulSet 能够保证这些 Pod 名字里的编号不变，那么 Service 里类似于 web-0.nginx.default.svc.cluster.local 这样的 DNS 记录也就不会变，而这条记录解析出来的 Pod 的 IP 地址，则会随着后端 Pod 的删除和再创建而自动更新。这当然是 Service 机制本身的能力，不需要 StatefulSet 操心。

**最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC**。这样，Kubernetes 就可以通过 Persistent Volume 机制为这个 PVC 绑定上对应的 PV，从而保证了每一个 Pod 都拥有一个独立的 Volume。

在这种情况下，即使 Pod 被删除，它所对应的 PVC 和 PV 依然会保留下来。所以当这个 Pod 被重新创建出来之后，Kubernetes 会为它找到同样编号的 PVC，挂载这个 PVC 对应的 Volume，从而获取到以前保存在 Volume 里的数据。

### DaemonSet

运行一个 Daemon Pod。 所以，这个 Pod 有如下三个特征：

1. 这个 Pod 运行在 Kubernetes 集群里的每一个节点（Node）上；
2. 每个节点上只有一个这样的 Pod 实例；
3. 当有新的节点加入 Kubernetes 集群后，该 Pod 会自动地在新节点上被创建出来；而当旧节点被删除后，它上面的 Pod 也相应地会被回收掉。

### Job

在 Job 对象中，负责并行控制的参数有两个：

1. spec.parallelism，它定义的是一个 Job 在任意时间最多可以启动多少个 Pod 同时运行；
2. spec.completions，它定义的是 Job 至少要完成的 Pod 数目，即 Job 的最小完成数.

Job Controller 实际上控制了，作业执行的**并行度**，以及总共需要完成的**任务数**这两个重要参数。而在实际使用时，你需要根据作业的特性，来决定并行度（parallelism）和任务数（completions）的合理取值。(就是根据预期数量和成功数量来判断Job是否完成)

使用 Job 对象的方法:

1. **外部管理器 +Job 模板。**

   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: process-item-$ITEM
     labels:
       jobgroup: jobexample
   spec:
     template:
       metadata:
         name: jobexample
         labels:
           jobgroup: jobexample
       spec:
         containers:
         - name: c
           image: busybox
           command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
         restartPolicy: Never
   ```

   

2. **拥有固定任务数目的并行 Job**。

   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: job-wq-1
   spec:
     completions: 8
     parallelism: 2
     template:
       metadata:
         name: job-wq-1
       spec:
         containers:
         - name: c
           image: myrepo/job-wq-1
           env:
           - name: BROKER_URL
             value: amqp://guest:guest@rabbitmq-service:5672
           - name: QUEUE
             value: job1
         restartPolicy: OnFailure
   ```

3. **指定并行度（parallelism），但不设置固定的 completions 的值。**

   ```yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: job-wq-2
   spec:
     parallelism: 2
     template:
       metadata:
         name: job-wq-2
       spec:
         containers:
         - name: c
           image: gcr.io/myproject/job-wq-2
           env:
           - name: BROKER_URL
             value: amqp://guest:guest@rabbitmq-service:5672
           - name: QUEUE
             value: job2
         restartPolicy: OnFailure
   ```

   spec.concurrencyPolicy 字段来定义具体的处理策略。比如：

   1. concurrencyPolicy=Allow，这也是默认情况，这意味着这些 Job 可以同时存在；
   2. concurrencyPolicy=Forbid，这意味着不会创建新的 Pod，该创建周期被跳过；
   3. concurrencyPolicy=Replace，这意味着新产生的 Job 会替换旧的、没有执行完的 Job。

#### CronJob

CronJob 是一个 Job 对象的控制器（Controller）
## 命令

| 命令                                               | 描述                                                         | 参数 | 参数作用 |
| -------------------------------------------------- | ------------------------------------------------------------ | ---- | -------- |
| kubectl apply                                      | 创建/更新（执行了一个**对原有 API 对象的 PATCH 操作**。）    |      |          |
| kubectl create  nginx-deployment.yaml --record     | 创建                                                         |      |          |
| kubectl replace -f nginx.yaml                      | 更新（使用新的 YAML 文件中的 API 对象，**替换原有的 API 对象**） |      |          |
| kubectl get deployments                            | 查看 deployments                                             |      |          |
| kubectl edit deployment/nginx-deployment           |                                                              |      |          |
| kubectl rollout status deployment/nginx-deployment | 查看上线状态                                                 |      |          |
| kubectl describe deployment nginx-deployment       | ``                                                           |      |          |
| kubectl set image                                  | 直接修改镜像                                                 |      |          |
| kubectl rollout undo deployment/nginx-deployment   | 把整个 Deployment 回滚到上一个版本                           |      |          |
| kubectl rollout history                            | **查看每次 Deployment 变更对应的版本**                       |      |          |
| kubectl rollout pause                              | Deployment 进入了一个“暂停”状态（这样就不会每次就该都触发创建一个新的**ReplicaSet**） |      |          |
| kubectl rollout resume                             | Deployment“恢复”                                             |      |          |
|                                                    |                                                              |      |          |



|

## yaml

### pod

```yaml
# yaml格式的pod定义文件完整内容：
apiVersion: v1       #必选，版本号，例如v1
kind: Pod       #必选，Pod
metadata:       #必选，元数据
  name: string       #必选，Pod名称
  namespace: string    #必选，Pod所属的命名空间
  labels:      #自定义标签（k,v）
    - name: string     #自定义标签名字
  annotations:       #自定义注释列表
    - name: string
spec:         #必选，Pod中容器的详细定义
  containers:      #必选，Pod中容器列表
  - name: string     #必选，容器名称
    image: string    #必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent] #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]    #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]     #容器的启动命令参数列表
    workingDir: string     #容器的工作目录
    volumeMounts:    #挂载到容器内部的存储卷配置
    - name: string     #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string    #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean    #是否为只读模式
    ports:       #需要暴露的端口库号列表
    - name: string     #端口号名称
      containerPort: int   #容器需要监听的端口号
      hostPort: int    #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string     #端口协议，支持TCP和UDP，默认TCP
    env:       #容器运行前需设置的环境变量列表
    - name: string     #环境变量名称
      value: string    #环境变量的值
    resources:       #资源限制和请求的设置
      limits:      #资源限制的设置
        cpu: string    #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string     #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:      #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string     #内存清楚，容器启动的初始可用数量
    livenessProbe:     #对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:      #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0  #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0   #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0    #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged:false
    restartPolicy: [Always | Never | OnFailure]#Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeSelector: obeject  #设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:    #Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork:false      #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:       #在该pod上定义共享存储卷列表
    - name: string     #共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}     #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string     #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string     #Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:      #类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap:     #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
          path: string

```

### Deployment 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3 # pod 数量，它的默认值是1“RollingUpdate” 是默认值。
  selector: # 定本 Deployment 的 Pod 标签选择算符的必需字段。必须匹配 .spec.template.metadata.labels，否则请求会被 API 拒绝。
    matchLabels:
      app: nginx
  template: # 和 Pod 的语法规则完全相同。 只是这里它是嵌套的，因此不需要 apiVersion 或 kind。
    metadata:
      labels: # .spec.selector 和 .metadata.labels 如果没有设置的话， 不会被默认设置为 .spec.template.metadata.labels，所以需要明确进行设置。
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
      restartPolicy: Always # 重新启动策略，默认是Always
  paused: false # 是用于暂停和恢复 Deployment 的可选布尔字段。 Deployment 在创建时是默认不会处于暂停状态。
  revisionHistoryLimit: 10 # (修订历史限制) 用来设定出于会滚目的所要保留的旧 ReplicaSet 数量。默认10
  minReadySeconds: 0 # (最短就绪时间)用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间， 只有超出这个时间 Pod 才被视为可用。
  progressDeadlineSeconds: 100 # (进度期限秒数) Deployment 进展失败 之前等待 Deployment 取得进展的秒数
  strategy:  # 策略指定用于用新 Pods 替换旧 Pods 的策略
    type: RollingUpdate # 可以是 “Recreate” 或 “RollingUpdate”。(Recreate，在创建新 Pods 之前，所有现有的 Pods 会被杀死。)(RollingUpdate时，采取 滚动更新的方式更新 Pods。你可以指定 maxUnavailable 和 maxSurge 来控制滚动更新 过程。)
    rollingUpdate:
      maxSurge: 1 # 除了 DESIRED 数量之外，在一次“滚动”中，Deployment 控制器还可以创建多少个新 Pod；(可以设置为百分比)
      maxUnavailable: 1 # 在一次“滚动”中，Deployment 控制器可以删除多少个旧 Pod
  revisionHistoryLimit: 10 # (清理策略)指定保留此 Deployment 的多少个旧有 ReplicaSet。其余的 ReplicaSet 将在后台被垃圾回收。 默认情况下，此值为 10。

```

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

### StatefulSet 

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template: # Pod 的语法规则完全相同。 只是这里它是嵌套的，因此不需要 apiVersion 或 kind。
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
          name: web

```


# nacos

nacos版本为1.1.3

## 服务注册

源码入口==spring-cloud-alibaba-nacos-discovery.ja==r里的spring.factorise文件里面的==NacosDiscoveryAutoConfiguration==

## NacosDiscoveryAutoConfiguration

注入三个Bean

### NacosServiceRegistry

### NacosRegistration

### NacosAutoServiceRegistration

注入相关配置，进行实例化启动，自动注入相关信息。

`NacosAutoServiceRegistration` 继承`AbstractAutoServiceRegistration` ,`AbstractAutoServiceRegistration` 继承`ApplicationListener`接口，将在spring 启动时调用

```java
》com.alibaba.cloud.nacos.registry.NacosAutoServiceRegistration  
 》org.springframework.cloud.client.serviceregistry.AbstractAutoServiceRegistration#onApplicationEvent
  》org.springframework.cloud.client.serviceregistry.AbstractAutoServiceRegistration#bind()
   》org.springframework.cloud.client.serviceregistry.AbstractAutoServiceRegistration#start()
	》org.springframework.cloud.client.serviceregistry.AbstractAutoServiceRegistration#register()
	 》com.alibaba.nacos.client.naming.NacosNamingService#registerInstance()
      》com.alibaba.nacos.client.naming.beat.BeatReactor#addBeatInfo
       》com.alibaba.nacos.client.naming.net.NamingProxy#sendBeat()
       》com.alibaba.nacos.client.naming.beat.BeatReactor.BeatTask#run()
      》com.alibaba.nacos.client.naming.net.NamingProxy#registerService()
       》com.alibaba.nacos.client.naming.net.NamingProxy#reqAPI()
    
    
public void onApplicationEvent(WebServerInitializedEvent event) {     this.bind(event); }
----------------------------------
@Deprecated
public void bind(WebServerInitializedEvent event) {
    ApplicationContext context = event.getApplicationContext();
    if (context instanceof ConfigurableWebServerApplicationContext) {
        if ("management".equals(((ConfigurableWebServerApplicationContext) context)
                                .getServerNamespace())) {
            return;
        }
    }
    this.port.compareAndSet(0, event.getWebServer().getPort());
    this.start();
}
----------------------------------
 public void start() {
    if (!isEnabled()) {
        if (logger.isDebugEnabled()) {
            logger.debug("Discovery Lifecycle disabled. Not starting");
        }
        return;
    }

    // only initialize if nonSecurePort is greater than 0 and it isn't already running
    // because of containerPortInitializer below
    if (!this.running.get()) {
        this.context.publishEvent(
            new InstancePreRegisteredEvent(this, getRegistration()));
        register();
        if (shouldRegisterManagement()) {
            registerManagement();
        }
        this.context.publishEvent(
            new InstanceRegisteredEvent<>(this, getConfiguration()));
        this.running.compareAndSet(false, true);
    }

}
-----------------------------
protected void register() {
    this.serviceRegistry.register(getRegistration());
}
--------------------------------
@Override
public void register(Registration registration) {

    if (StringUtils.isEmpty(registration.getServiceId())) {
        log.warn("No service to register for nacos client...");
        return;
    }

    String serviceId = registration.getServiceId();

    Instance instance = getNacosInstanceFromRegistration(registration);

    try {
        namingService.registerInstance(serviceId, instance);
        log.info("nacos registry, {} {}:{} register finished", serviceId,
                 instance.getIp(), instance.getPort());
    }
    catch (Exception e) {
        log.error("nacos registry, {} register failed...{},", serviceId,
                  registration.toString(), e);
    }
}
----------------------------
@Override
public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {

    if (instance.isEphemeral()) {
        BeatInfo beatInfo = new BeatInfo();
        beatInfo.setServiceName(NamingUtils.getGroupedName(serviceName, groupName));
        beatInfo.setIp(instance.getIp());
        beatInfo.setPort(instance.getPort());
        beatInfo.setCluster(instance.getClusterName());
        beatInfo.setWeight(instance.getWeight());
        beatInfo.setMetadata(instance.getMetadata());
        beatInfo.setScheduled(false);
        long instanceInterval = instance.getInstanceHeartBeatInterval();
        beatInfo.setPeriod(instanceInterval == 0 ? DEFAULT_HEART_BEAT_INTERVAL : instanceInterval);

        beatReactor.addBeatInfo(NamingUtils.getGroupedName(serviceName, groupName), beatInfo);
    }

    serverProxy.registerService(NamingUtils.getGroupedName(serviceName, groupName), groupName, instance);
}
----------------------------------
    ===================================
    // 添加定时任务（心跳连接）
 	public void addBeatInfo(String serviceName, BeatInfo beatInfo) {
        NAMING_LOGGER.info("[BEAT] adding beat: {} to beat map.", beatInfo);
        dom2Beat.put(buildKey(serviceName, beatInfo.getIp(), beatInfo.getPort()), beatInfo);
        executorService.schedule(new BeatTask(beatInfo), 0, TimeUnit.MILLISECONDS);
        MetricsMonitor.getDom2BeatSizeMonitor().set(dom2Beat.size());
    }
 	===================================
    BeatTask（任务对象）
	@Override
    public void run() {
            if (beatInfo.isStopped()) {
                return;
            }
            long result = serverProxy.sendBeat(beatInfo);
            long nextTime = result > 0 ? result : beatInfo.getPeriod();
            executorService.schedule(new BeatTask(beatInfo), nextTime, TimeUnit.MILLISECONDS);
        }
 =====================================
 	public long sendBeat(BeatInfo beatInfo) {
        try {
            if (NAMING_LOGGER.isDebugEnabled()) {
                NAMING_LOGGER.debug("[BEAT] {} sending beat to server: {}", namespaceId, beatInfo.toString());
            }
            Map<String, String> params = new HashMap<String, String>(4);
            params.put("beat", JSON.toJSONString(beatInfo));
            params.put(CommonParams.NAMESPACE_ID, namespaceId);
            params.put(CommonParams.SERVICE_NAME, beatInfo.getServiceName());
            // 调用server的实例发送心跳接口（心跳访问地址：/v1/ns/instance/beat）
            String result = reqAPI(UtilAndComs.NACOS_URL_BASE + "/instance/beat", params, HttpMethod.PUT);
            JSONObject jsonObject = JSON.parseObject(result);

            if (jsonObject != null) {
                return jsonObject.getLong("clientBeatInterval");
            }
        } catch (Exception e) {
            NAMING_LOGGER.error("[CLIENT-BEAT] failed to send beat: " + JSON.toJSONString(beatInfo), e);
        }
        return 0L;
    }
=====================================

----------------------------------
public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {

    NAMING_LOGGER.info("[REGISTER-SERVICE] {} registering service {} with instance: {}",
                       namespaceId, serviceName, instance);

    final Map<String, String> params = new HashMap<String, String>(9);
    params.put(CommonParams.NAMESPACE_ID, namespaceId);
    params.put(CommonParams.SERVICE_NAME, serviceName);
    params.put(CommonParams.GROUP_NAME, groupName);
    params.put(CommonParams.CLUSTER_NAME, instance.getClusterName());
    params.put("ip", instance.getIp());
    params.put("port", String.valueOf(instance.getPort()));
    params.put("weight", String.valueOf(instance.getWeight()));
    params.put("enable", String.valueOf(instance.isEnabled()));
    params.put("healthy", String.valueOf(instance.isHealthy()));
    params.put("ephemeral", String.valueOf(instance.isEphemeral()));

	// 拼接注册地址（/nacos/v1/ns/instance）
    reqAPI(UtilAndComs.NACOS_URL_INSTANCE, params, HttpMethod.POST);
}
----------------------------------
```

### Server端

==/nacos/v1/ns/instance==(Post 将客户端注册到服务中心)

```java
 @CanDistro
 @PostMapping
 @Secured(parser = NamingResourceParser.class, action = ActionTypes.WRITE)
 public String register(HttpServletRequest request) throws Exception {

String serviceName = WebUtils.required(request, CommonParams.SERVICE_NAME);
String namespaceId = WebUtils.optional(request, CommonParams.NAMESPACE_ID, Constants.DEFAULT_NAMESPACE_ID);

serviceManager.registerInstance(namespaceId, serviceName, parseInstance(request));
return "ok";
}
-------------------------------------------------
public void registerInstance(String namespaceId, String serviceName, Instance instance) throws NacosException {

    createEmptyService(namespaceId, serviceName, instance.isEphemeral());

    Service service = getService(namespaceId, serviceName);

    if (service == null) {
        throw new NacosException(NacosException.INVALID_PARAM,
                                 "service not found, namespace: " + namespaceId + ", service: " + serviceName);
    }
	// 添加server实例
    addInstance(namespaceId, serviceName, instance.isEphemeral(), instance);
}
-------------------------------------------------
// createEmptyService(namespaceId, serviceName, instance.isEphemeral());
public void createServiceIfAbsent(String namespaceId, String serviceName, boolean local, Cluster cluster) throws NacosException {
    Service service = getService(namespaceId, serviceName);
    if (service == null) {

        Loggers.SRV_LOG.info("creating empty service {}:{}", namespaceId, serviceName);
        service = new Service();
        service.setName(serviceName);
        service.setNamespaceId(namespaceId);
        service.setGroupName(NamingUtils.getGroupName(serviceName));
        // now validate the service. if failed, exception will be thrown
        service.setLastModifiedMillis(System.currentTimeMillis());
        service.recalculateChecksum();
        if (cluster != null) {
            cluster.setService(service);
            service.getClusterMap().put(cluster.getName(), cluster);
        }
        service.validate();

        putServiceAndInit(service);
        if (!local) {
            addOrReplaceService(service);
        }
    }
}

private void putServiceAndInit(Service service) throws NacosException {
    // (重点)客户端信息存储结构：  Map<namespace, Map<group::serviceName, Service>> serviceMap = new ConcurrentHashMap<>();
    // 如果是为空，就是只是构造一个空的结构，还未注入实例
    putService(service);
    service.init();
    consistencyService.listen(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), true), service);
    consistencyService.listen(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), false), service);
    Loggers.SRV_LOG.info("[NEW-SERVICE] {}", service.toJSON());
}

public void init() {
    HealthCheckReactor.scheduleCheck(clientBeatCheckTask);

    for (Map.Entry<String, Cluster> entry : clusterMap.entrySet()) {
        entry.getValue().setService(this);
        entry.getValue().init();
    }
}

public static void scheduleCheck(ClientBeatCheckTask task) {
    // 执行定时任务
    futureMap.putIfAbsent(task.taskKey(), EXECUTOR.scheduleWithFixedDelay(task, 5000, 5000, TimeUnit.MILLISECONDS));
}


@Override
public void run() {
    try {
        if (!getDistroMapper().responsible(service.getName())) {
            return;
        }

        if (!getSwitchDomain().isHealthCheckEnabled()) {
            return;
        }

        List<Instance> instances = service.allIPs(true);

        // first set health status of instances:
        for (Instance instance : instances) {
            // 先判断当前时间与上次心跳时间的间隔是否大于超时时间。如果实例已经超时，且为被标记，且健康状态为健康，则将健康状态设置为不健康，同时发布状态变化的事件。
            if (System.currentTimeMillis() - instance.getLastBeat() > instance.getInstanceHeartBeatTimeOut()) {
                if (!instance.isMarked()) {
                    if (instance.isHealthy()) {
                        instance.setHealthy(false);
                        Loggers.EVT_LOG.info("{POS} {IP-DISABLED} valid: {}:{}@{}@{}, region: {}, msg: client timeout after {}, last beat: {}",
                                             instance.getIp(), instance.getPort(), instance.getClusterName(), service.getName(),
                                             UtilsAndCommons.LOCALHOST_SITE, instance.getInstanceHeartBeatTimeOut(), instance.getLastBeat());
                        getPushService().serviceChanged(service);
                        SpringContext.getAppContext().publishEvent(new InstanceHeartbeatTimeoutEvent(this, instance));
                    }
                }
            }
        }

        if (!getGlobalConfig().isExpireInstance()) {
            return;
        }

        // 如果实例已经被标记则跳出循环。如果未标记，同时当前时间与上次心跳时间的间隔大于删除IP时间，则将对应的实例删除。
        for (Instance instance : instances) {

            if (instance.isMarked()) {
                continue;
            }

            // 默认30秒
            if (System.currentTimeMillis() - instance.getLastBeat() > instance.getIpDeleteTimeout()) {
                // delete instance
                Loggers.SRV_LOG.info("[AUTO-DELETE-IP] service: {}, ip: {}", service.getName(), JSON.toJSONString(instance));
                deleteIP(instance);
            }
        }

    } catch (Exception e) {
        Loggers.SRV_LOG.warn("Exception while processing client beat time out.", e);
    }

}
```

==/nacos/v1/ns/instance/beat== 发送实例心跳



参考： [文章总结](https://note.youdao.com/ynoteshare1/index.html?id=17c68958637d60582e9c473f69f04aa5&type=note)
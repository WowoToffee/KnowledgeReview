# nacos

nacos版本为1.1.3

## 服务注册

源码入口 ==spring-cloud-alibaba-nacos-discovery.jar==里的spring.factorise文件里面的==NacosDiscoveryAutoConfiguration==

## NacosDiscoveryAutoConfiguration

创建三个Bean

### NacosServiceRegistry



### NacosRegistration

### NacosAutoServiceRegistration

注入相关配置，进行实例化启动，自动注入相关信息。

`NacosAutoServiceRegistration` 继承`AbstractAutoServiceRegistration` ,`AbstractAutoServiceRegistration` 继承`ApplicationListener`接口，将在spring 启动时调用

```java

public void onApplicationEvent(WebServerInitializedEvent event) {
	this.bind(event);
}
```



参考：
[文章总结](https://note.youdao.com/ynoteshare1/index.html?id=17c68958637d60582e9c473f69f04aa5&type=note)

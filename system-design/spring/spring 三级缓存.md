# spring 三级缓存

## spring 缓存机制

![](https://img2020.cnblogs.com/blog/2027777/202008/2027777-20200822213542986-1450430873.png)

![](https://img2020.cnblogs.com/blog/2027777/202008/2027777-20200822224312938-1691750102.png)

### 一级缓存singletonObjects

用于保存BeanName和创建bean实例之间的关系,beanName -> bean instance

> private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);

### 二级缓存earlySingletonObjects

保存提前曝光的单例bean对象

> private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

### 三级缓存singletonFactories

保存beanName和创建bean实例之间的关系,与singletonObjects不同的地方在于，当一个单例bean被放到这里面后，bean在创建过程中，可以通过getBean方法获取到,目的是用来检测循环引用

> private final Map<String, Object> singletonFactories = new HashMap(16);

在创建bean的时候，首先从缓存中获取单例的bean，这个缓存就是**singletonObjects**，如果获取不到且bean正在创建中，就再从**earlySingletonObjects**中获取，如果还是获取不到且允许从singletonFactories中通过getObject拿到对象，就从**singletonFactories**中获取，如果获取到了就存入**earlySingletonObjects**并从**singletonFactories**中移除。

## 为什么要使用三级缓存?

### 一级缓存

首先我们看看一级缓存行不行，如果只留第一级缓存，那么单例的Bean都存在singletonObjects 中，Spring循环依赖主要基于Java引用传递，当获取到对象时，对象的field或者属性可以延后设置，理论上可以，但是如果延后设置出了问题，就会导致完整的Bean和不完整的Bean都在一级缓存中，这个引用时就有空指针的可能，所以一级缓存不行，至少要有singletonObjects 和earlySingletonObjects 两级。


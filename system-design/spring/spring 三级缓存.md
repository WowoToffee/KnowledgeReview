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

### 一级缓存不行？

首先我们看看一级缓存行不行，如果只留第一级缓存，那么单例的Bean都存在singletonObjects 中，Spring循环依赖主要基于Java引用传递，当获取到对象时，对象的field或者属性可以延后设置，理论上可以，但是如果延后设置出了问题，就会导致完整的Bean和不完整的Bean都在一级缓存中，这个引用时就有空指针的可能，所以一级缓存不行，至少要有singletonObjects 和earlySingletonObjects 两级。

### 二级缓存不行？

如果`对象A`和`对象B`循环依赖，且都有代理的话，那创建的顺序就是

![image-20200706161709829](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vd3hfY2MzNDdiZTY5Ni9ibG9nSW1hZ2UvcmF3L21hc3Rlci9pbWFnZS0yMDIwMDcwNjE2MTcwOTgyOS5wbmc?x-oss-process=image/format,png)

1. `A半成品`加入`第三级缓存`
2. `A`填充属性注入`B` -> 创建`B对象` -> `B半成品`加入`第三级缓存`
3. `B`填充属性注入`A` -> 创建`A代理对象`，从`第三级缓存`移除`A对象`，`A代理对象`加入`第二级缓存`（此时`A`还是半成品，`B`注入的是`A代理对象`）
4. 创建`B代理对象`（此时`B`是完成品） -> 从`第三级缓存`移除`B对象`，`B代理对象`加入`第一级缓存`
5. `A半成品`注入`B代理对象`
6. 从`第二级缓存`移除`A代理对象`，`A代理对象`加入`第一级缓存`

如果只是有两级缓存，二级缓存存代理对象，那么就会丢失ObjectFactory A,需要重新创建。

如果只是有两级缓存，二级缓存存ObjectFactory A，那么在后面如果有一个对象C 也同样依赖A,需要重新创建代理对象A，这样就会出现两个代理对象A。




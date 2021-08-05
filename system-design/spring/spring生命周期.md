# spring 生命周期

## InitializingBean接口

**通过让Bean实现==InitializingBean==（定义初始化逻辑）**该接口将重写==afterPropertiesSet()== 方法



## DisposableBean接口

通过让Bean实现**==DisposableBean==（定义销毁逻辑）**该接口将重写==destroy()== 方法



## @PostConstruct注解

在bean创建完成并赋值完成后实例化之前，进行的初始化操作



## @PreDestroy注解

在容器销毁bean之前，通知进行销毁操作



## BeanDefinitionRegistryPostProcessor

**执行时机:所有的bean定义信息将要被加载到容器中，Bean实例还没有被初始化**



## BeanPostProcessor：Bean的后置处理器

**执行时间:所有的Bean定义信息已经加载到容器中，但是Bean实例还没有被初始化.**

### postProcessBeforeInitialization:在初始化之前工作

### postProcessAfterInitialization:在初始化之后工作



1. 构造（对象创建）： 
   1. 单实例：在容器启动的时候创建对象
   2. 多实例：在每次获取的时候创建对象
2. BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry
3. BeanPostProcessor.postProcessBeforeInitialization
4. 初始化：对象创建完成，并赋值好，调用初始化方法。。。
5. BeanPostProcessor.postProcessAfterInitialization
6. 销毁：
   1. 单实例：容器关闭的时候
   2. 多实例：容器不会管理这个bean；容器不会调用销毁方法



- 如果涉及到一些属性值 利用 `set()` 方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()` 方法，传入 `ClassLoader` 对象的实例。
- 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanClassLoader()` 方法，传入 `ClassLoader` 对象的实例。

![Spring Bean 生命周期](https://res-static.hc-cdn.cn/fms/img/d78ca01af24b7ec38aa705e22b8a0ef91603772016992)
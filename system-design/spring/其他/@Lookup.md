# @Lookup

## @Lockup简单介绍

- 用于解决单例依赖原型Bean，原型Bean不生效的情况。
- 核心思路是生成生成代理对象，执行代理对象的方法。
- 生成的代理对象是单例Bean去生成代理对象，不是原型Bean！！！
- 本文源码位置之寻找含有@Lockup注解的方法：org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.determineCandidateConstructors(Class<?>, String)
- 本文源码位置之生成代理对象源码：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateBean(String, RootBeanDefinition)
- 本文源码位置之执行代理对象源码：- 本文源码位置之生成代理对象源码：org.springframework.beans.factory.support.CglibSubclassingInstantiationStrategy.LookupOverrideMethodInterceptor(内部类).intercept(Object obj, Method method, Object[] args, MethodProxy mp)

## 使用方式

```
@Component
public class Tset1Service {
    public void printClass() {
        System.out.println(a());
    }

    // 配置具体的原型Bean
    @Lookup("tset2Service")
    public ClassB a() {
        return null;
    }
}1.2.3.4.5.6.7.8.9.10.11.12.
```

## 注入点源码分析，找到含有@Lockup注解的方法

```java
public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, final String beanName)
    throws BeanCreationException {

  // Let's check for lookup methods here...
  if (!this.lookupMethodsChecked.contains(beanName)) {
    // 判断beanClass是不是java.开头的类，比如String
    if (AnnotationUtils.isCandidateClass(beanClass, Lookup.class)) {
      try {
        Class<?> targetClass = beanClass;
        do {
          // 遍历targetClass中的method，查看是否写了@Lookup方法
          ReflectionUtils.doWithLocalMethods(targetClass, method -> {
            Lookup lookup = method.getAnnotation(Lookup.class);
            if (lookup != null) {
              Assert.state(this.beanFactory != null, "No BeanFactory available");

              // 将当前method封装成LookupOverride并设置到BeanDefinition的methodOverrides中
              LookupOverride override = new LookupOverride(method, lookup.value());
              try {
                RootBeanDefinition mbd = (RootBeanDefinition)
                    this.beanFactory.getMergedBeanDefinition(beanName);
                mbd.getMethodOverrides().addOverride(override);
              }
              catch (NoSuchBeanDefinitionException ex) {
                throw new BeanCreationException(beanName,
                    "Cannot apply @Lookup to beans without corresponding bean definition");
              }
            }
          });
          targetClass = targetClass.getSuperclass();
        }
        while (targetClass != null && targetClass != Object.class);

      }
      catch (IllegalStateException ex) {
        throw new BeanCreationException(beanName, "Lookup method resolution failed", ex);
      }
    }
    this.lookupMethodsChecked.add(beanName);
  }
  ......
}
```

## 生成代理对象的方法

```java
protected BeanWrapper instantiateBean(String beanName, RootBeanDefinition mbd) {
  try {
    Object beanInstance;
    if (System.getSecurityManager() != null) {
      beanInstance = AccessController.doPrivileged(
          (PrivilegedAction<Object>) () -> getInstantiationStrategy().instantiate(mbd, beanName, this),
          getAccessControlContext());
    }
    else {
      // 策略模式，默认是CglibSubclassingInstantiationStrategy
      beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, this);
    }
    BeanWrapper bw = new BeanWrapperImpl(beanInstance);
    initBeanWrapper(bw);
    return bw;
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
        mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
  }
}

/**
 * 实例化
 */
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
  // Don't override the class with CGLIB if no overrides.
  // 判断当前BeanDefinition对应的beanClass中是否存在@Lookup的方法
  if (!bd.hasMethodOverrides()) {
    ......
  }
  else {
    // Must generate CGLIB subclass.
    // 如果存在@Lookup，则会生成一个代理对象
    return instantiateWithMethodInjection(bd, beanName, owner);
  }
}

/**
 * 生成代理对象的重载方法
 */
protected Object instantiateWithMethodInjection(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
  return instantiateWithMethodInjection(bd, beanName, owner, null);
}

/**
 * 生成代理对象的重载方法
 */
protected Object instantiateWithMethodInjection(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner,
    @Nullable Constructor<?> ctor, Object... args) {

  // Must generate CGLIB subclass...
  // 调用Cglib去生成一个代理对象
  return new CglibSubclassCreator(bd, owner).instantiate(ctor, args);
}

/**
 * 生成代理对象
 */
public Object instantiate(@Nullable Constructor<?> ctor, Object... args) {
  Class<?> subclass = createEnhancedSubclass(this.beanDefinition);
  Object instance;
  // 生成代理对象
  if (ctor == null) {
    instance = BeanUtils.instantiateClass(subclass);
  }
  else {
    try {
      Constructor<?> enhancedSubclassConstructor = subclass.getConstructor(ctor.getParameterTypes());
      instance = enhancedSubclassConstructor.newInstance(args);
    }
    catch (Exception ex) {
      throw new BeanInstantiationException(this.beanDefinition.getBeanClass(),
          "Failed to invoke constructor for CGLIB enhanced subclass [" + subclass.getName() + "]", ex);
    }
  }
  // 代理对象具体的执行逻辑
  // SPR-10785: set callbacks directly on the instance instead of in the
  // enhanced class (via the Enhancer) in order to avoid memory leaks.
  Factory factory = (Factory) instance;
  factory.setCallbacks(new Callback[] {NoOp.INSTANCE,
      new LookupOverrideMethodInterceptor(this.beanDefinition, this.owner),
      new ReplaceOverrideMethodInterceptor(this.beanDefinition, this.owner)});
  return instance;
}
```

## 代理对象具体的执行逻辑：直接调用getBean

```java
public Object intercept(Object obj, Method method, Object[] args, MethodProxy mp) throws Throwable {
  // Cast is safe, as CallbackFilter filters are used selectively.
  // 当前执行的方法上有没有@Lockup注解
  LookupOverride lo = (LookupOverride) getBeanDefinition().getMethodOverrides().getOverride(method);
  Assert.state(lo != null, "LookupOverride not found");
  // 得到所有参数
  Object[] argsToUse = (args.length > 0 ? args : null);  // if no-arg, don't insist on args at all
  if (StringUtils.hasText(lo.getBeanName())) {
    // 调用getBean
    Object bean = (argsToUse != null ? this.owner.getBean(lo.getBeanName(), argsToUse) :
        this.owner.getBean(lo.getBeanName()));
    // Detect package-protected NullBean instance through equals(null) check
    // 直接返回Bean，不去执行具体的方法
    return (bean.equals(null) ? null : bean);
  }
  else {
    // Find target bean matching the (potentially generic) method return type
    ResolvableType genericReturnType = ResolvableType.forMethodReturnType(method);
    return (argsToUse != null ? this.owner.getBeanProvider(genericReturnType).getObject(argsToUse) :
        this.owner.getBeanProvider(genericReturnType).getObject());
  }
}
```


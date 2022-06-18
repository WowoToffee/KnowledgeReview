# spring为什么使用三级缓存？直接一级？或者只用二级？

# 自动注入（@Autowried ）

能完成自动注入的AutowireMode 类型有 0.==on== 1.==bytype== 2.==byname== 3.==constucre==

1. @Autowried  的  AutowireMode数值其实0（no）

   它之所能完成属性的装配

2. 构造函数

```java 
private Z z;

public CompentA(Z z) {
this.z = z;
}
```

这种方式注入属性 AutowireMode数值也是0（no）

它之所能完成属性的装配是因为spring 在 ==createBeanInstance== 的时候进行了兼容处理

```java
// Need to determine the constructor...
Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
if (ctors != null ||
    mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
    mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
    return autowireConstructor(beanName, mbd, ctors, args);
}
```

# spring MVC请求流程

1. 用户发送URL至前端控制器上（DispatcherServlet）

2. 前端控制器(DispatcherServlet)收到请求后调用处理器映射器（HandlerMapping）

3. 处理器映射器（HandlerMapping）根据请求的URL找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。

4. 前端控制器（DispatcherServlet）根据处理器Headler获取处理适配器（HandlerAdapter）执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作

5. 执行处理器Handler（Controller，也叫做页面控制器）

6. Handler执行完成返回ModelAndView

7. 处理适配器（HandlerAdapter）将Handler执行结果ModeAndView返回给前端控制器（DispatcherServlet）

8. 前端控制器（DispatcherServlet）将ModeAndView传给视图解析器（ViewReslover）

9. ViewReslover解析后返回具体View

10. DispatcherServlet对view进行渲染视图（将模型数据model填充至视图中）

11. DispatcherServlet响应客户

    


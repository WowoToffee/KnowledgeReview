[1.Spring framework 核心模块有哪些？](#1)

[2.Spring framework的优势和不足是什么？](#2)

[3.为什么说ObjectFactory提供的是延迟依赖查找？](#3)

[4.什么是IOC？](#4)

[5.依赖查找和依赖注入的区别](#5)

[6.Spring作为IOC容器有什么优势？](#6)

[7.什么是Spring IOC容器](#7)

[8.BeanFactory 与FactoryBean ](#8)

[9.Spring IOC容器启动时做了那些准备 ](#9)



















<span id="1">1.Spring framework 核心模块有哪些？</span>

<span id="2">2.Spring framework的优势和不足是什么？</span>

<span id="3">3.为什么说ObjectFactory提供的是延迟依赖查找？</span>

<span id="4">4.什么是IOC？</span>

```IOC是控制反转，类似于好莱坞原则，主要有依赖查找和依赖注入实现```

<span id="5">5.依赖查找和依赖注入的区别</span>

```依赖查找是主动或手动的依赖查找方式，通常需要依赖容器或标准API实现，而依赖注入则是手动或自动依赖绑定的方法，无需依赖特定的容器和API```

<span id="6">6.Spring作为IOC容器有什么优势？</span>

<span id="7">7.什么是Spring IOC容器</span>

<span id="8">8.BeanFactory 与FactoryBean </span>

``````
BeanFactory 是IOC底层容器
FactoryBean 是创建Bean的一种方式，帮助实现复杂的初始化逻辑
``````

<span id="9">9.Spring IOC容器启动时做了那些准备  </span>

IOC设置元信息读取和解析，IOC容器生命周期，spring事件发布，国际化等





BeanDefinition元信息

| 属性                    | 说明                                          |
| ----------------------- | --------------------------------------------- |
| Class                   | Bean 全类名，必须是具体类，不能用抽象类和接口 |
| name                    | Bean的名称或者ID                              |
| Scope                   | Bean的作用域（如：singleton,prototype等）     |
| Constructor arguments   | Bean的构造器参数（用于依赖注入）              |
| Properties              | Bean的属性设置（用于依赖注入）                |
| Autowiring  mode        | Bean自动绑定模式（如：通过名ByName）          |
| Lazy initalization mode | Bean延迟初始化模式（延迟和非延迟）            |
| initalization mode      | Bean初始化回调方法名称                        |
| Destruction method      | Bean销毁回调方法名称                          |


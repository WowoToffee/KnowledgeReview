# spring 源码解析



## BeanDefinition



## DefaultListableBeanFactory



## spring IOC源码解析

```java
// 管理注册的Bean
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
    // 1. 初始化bean读取器和扫描器;
    this();
    //2.注册bean配置类
    this.register(annotatedClasses);
    //3.刷新上下文
    this.refresh();
}
```

### 1.this（）

```java
public AnnotationConfigApplicationContext() {
    //在IOC容器中初始化一个 注解bean读取器AnnotatedBeanDefinitionReader
   this.reader = new AnnotatedBeanDefinitionReader(this);
   //在IOC容器中初始化一个 按类路径扫描注解bean的 扫描器
   this.scanner = new ClassPathBeanDefinitionScanner(this);
}

----------------------------------------------------------------------
// 为bean定义读取器赋值
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}

----------------------------------------------------------------------
private static Environment getOrCreateEnvironment(BeanDefinitionRegistry registry) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    if (registry instanceof EnvironmentCapable) {
        return ((EnvironmentCapable) registry).getEnvironment();
    }
    return new StandardEnvironment();
}

----------------------------------------------------------------------
    创建一个条件计算器对象
public ConditionContextImpl(BeanDefinitionRegistry registry, Environment environment, ResourceLoader resourceLoader) {
			this.registry = registry;
			this.beanFactory = deduceBeanFactory(registry);
			this.environment = (environment != null ? environment : deduceEnvironment(registry));
			this.resourceLoader = (resourceLoader != null ? resourceLoader : deduceResourceLoader(registry));
}

----------------------------------------------------------------------
AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);为容器中注册系统的bean定义信息

public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, Object source) {
        
        //获取一个IOC容器
		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
		if (beanFactory != null) {
			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
			}
			
			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
			}
		}

		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<BeanDefinitionHolder>(8);
        
        //注册一个配置类解析器的bean定义（ConfigurationClassPostProcessor）
		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        
        //设置AutoWired注解解析器的bean定义信息
		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        
        // 注册解析@Required 注解的处理器
		if (!registry.containsBeanDefinition(REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, REQUIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        
        //检查是否支持JSR250规范，如何支持注册 解析JSR250规范的注解
		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        
        //检查是否支持jpa，若支持注册解析jpa规范的注解
		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}
        
        //注册解析@EventListener的注解
		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}
        
		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
	}

---------------------------------------------------------------------
创建类路径下的bean定义扫描器
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
			Environment environment, ResourceLoader resourceLoader) {

    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    this.registry = registry;

    //使用默认的扫描规则
    if (useDefaultFilters) {
        registerDefaultFilters();
    }
    //设置环境
    setEnvironment(environment);
    //设置资源加载器
    setResourceLoader(resourceLoader);
}

----------------------------------------------------------------
    registerDefaultFilters注册包扫描默认的规则 
    protected void registerDefaultFilters() {
	    //可以谢谢@Compent  @Service @Repository @Controller @Aspectj
		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
		try {
		    //jsr250规范的组件
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
			logger.debug("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
		}
		try {
		    //可以支持jsr330的注解
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
			logger.debug("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-330 API not available - simply skip.
		}
	}
	
----------------------------------------------------------------
```

### 2.register(annotatedClasses)

注册bean配置类, AnnotationConfigApplicationContext容器通过AnnotatedBeanDefinitionReader的register方法实现注解bean的读取，具体源码如下：

 `AnnotationConfigApplicationContext.java`中register方法

```java
//按指定bean配置类读取bean
public void register(Class<?>... annotatedClasses) {
   for (Class<?> annotatedClass : annotatedClasses) {
      registerBean(annotatedClass);
   }
}

public void registerBean(Class<?> annotatedClass) {
   doRegisterBean(annotatedClass, null, null, null);
}

//核心实现逻辑
<T> void doRegisterBean(Class<T> annotatedClass, @Nullable Supplier<T> instanceSupplier, @Nullable String name,
      @Nullable Class<? extends Annotation>[] qualifiers, BeanDefinitionCustomizer... definitionCustomizers) {
    //将Bean配置类信息转成容器中AnnotatedGenericBeanDefinition数据结构, AnnotatedGenericBeanDefinition继承自BeanDefinition作用是定义一个bean的数据结构，下面的getMetadata可以获取到该bean上的注解信息
   AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);
    //@Conditional装配条件判断是否需要跳过注册
   if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
      return;
   }
   //@param instanceSupplier a callback for creating an instance of the bean
   //设置回调
   abd.setInstanceSupplier(instanceSupplier);
   //解析bean作用域(单例或者原型)，如果有@Scope注解，则解析@Scope，没有则默认为singleton
   ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
　　//作用域写回BeanDefinition数据结构, abd中缺损的情况下为空，将默认值singleton重新赋值到abd
   abd.setScope(scopeMetadata.getScopeName());
　　//生成bean配置类beanName
   String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
   //通用注解解析到abd结构中，主要是处理Lazy, primary DependsOn, Role ,Description这五个注解
   AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
　　//@param qualifiers specific qualifier annotations to consider, if any, in addition to qualifiers at the bean class level
　　// @Qualifier特殊限定符处理，
   if (qualifiers != null) {
      for (Class<? extends Annotation> qualifier : qualifiers) {
         if (Primary.class == qualifier) {
    // 如果配置@Primary注解，则设置当前Bean为自动装配autowire时首选bean
            abd.setPrimary(true);
         }
　　else if (Lazy.class == qualifier) {
　　//设置当前bean为延迟加载
            abd.setLazyInit(true);
         }
         else {
　　　　　　//其他注解，则添加到abd结构中
            abd.addQualifier(new AutowireCandidateQualifier(qualifier));
         }
      }
   }
　　//自定义bean注册，通常用在applicationContext创建后，手动向容器中一lambda表达式的方式注册bean,
　　//比如：applicationContext.registerBean(UserService.class, () -> new UserService());
   for (BeanDefinitionCustomizer customizer : definitionCustomizers) {
　　　　 //自定义bean添加到BeanDefinition
      customizer.customize(abd);
   }
   //根据beanName和bean定义信息封装一个beanhold,heanhold其实就是一个 beanname和BeanDefinition的映射
   BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
　　//创建代理对象
   definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
　　// BeanDefinitionReaderUtils.registerBeanDefinition 内部通过DefaultListableBeanFactory.registerBeanDefinition(String beanName, BeanDefinition beanDefinition)按名称将bean定义信息注册到容器中，
　　// 实际上DefaultListableBeanFactory内部维护一个Map<String, BeanDefinition>类型变量beanDefinitionMap，用于保存注bean定义信息（beanname 和 beandefine映射）
   BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}

```

### 3. refresh()刷新上下文

 refresh方法在AbstractApplicationContext容器中实现，refresh()方法的作用加载或者刷新当前的配置信息，如果已经存在spring容器，则先销毁之前的容器，重新创建spring容器，载入bean定义，完成容器初始化工作.

AbstractApplicationContext.java中refresh方法的实现代码如下：

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      //1.刷新上下文前的预处理
      prepareRefresh();

      //2.获取刷新后的内部Bean工厂
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      //3.BeanFactory的预准备工作
      prepareBeanFactory(beanFactory);

      try {
         // BeanFactory准备工作完成后，可以做一些后置处理工作，
　　　　  // 4.空方法，用于在容器的子类中扩展
         postProcessBeanFactory(beanFactory);

         // 5. 执行BeanFactoryPostProcessor的方法，BeanFactory的后置处理器，在BeanFactory标准初始化之后执行的
         invokeBeanFactoryPostProcessors(beanFactory);

         // 6. 注册BeanPostProcessor（Bean的后置处理器）,用于拦截bean创建过程
         registerBeanPostProcessors(beanFactory);

         // 7. 初始化MessageSource组件（做国际化功能；消息绑定，消息解析）
         initMessageSource();

         // 8. 初始化事件派发器
         initApplicationEventMulticaster();

         // 9.空方法，可以用于子类实现在容器刷新时自定义逻辑
         onRefresh();

         // 10. 注册时间监听器，将所有项目里面的ApplicationListener注册到容器中来
         registerListeners();

         // 11. 初始化所有剩下的单实例bean,单例bean在初始化容器时创建，原型bean在获取时（getbean）时创建
         finishBeanFactoryInitialization(beanFactory);

         // 12. 完成BeanFactory的初始化创建工作，IOC容器就创建完成；
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

#### 3.1  prepareRefresh()：

刷新上线文前的预处理 AbstractApplicationContext. prepareRefresh ()方法:

```java
protected void prepareRefresh() {
　　//设置容器启动时间
   this.startupDate = System.currentTimeMillis();
　　//启动标识
   this.closed.set(false);
   this.active.set(true);

   if (logger.isInfoEnabled()) {
      logger.info("Refreshing " + this);
   }

   //空方法，用于子容器自定义个性化的属性设置方法
   initPropertySources();
   //检验属性的合法等
   getEnvironment().validateRequiredProperties();

   //保存容器中的一些早期的事件
   this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### 3.2 AbstractApplicationContext. obtainFreshBeanFactory()

获取刷新后的内部Bean工厂，obtainFreshBeanFactory方法为内部bean工厂重新生成id，并返回bean工厂

AbstractApplicationContext. obtainFreshBeanFactory()方法

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
　　//为beanfactory生成唯一序列化id，beanfactory已经在GenericApplicationContext构造函数中初始化了，refreshBeanFactory的逻辑在AbstractApplicationContext的实现类GenericApplicationContext中
 refreshBeanFactory();
　　 //获取beanfactory
   ConfigurableListableBeanFactory beanFactory = getBeanFactory();
   if (logger.isDebugEnabled()) {
      logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
   }
   return beanFactory;
}
---------------------------------------------------------------------------
protected final void refreshBeanFactory() throws IllegalStateException {
   if (!this.refreshed.compareAndSet(false, true)) {
      throw new IllegalStateException(
            "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
   }
　　//生成一个序列化id
   this.beanFactory.setSerializationId(getId());
}
```

#### 3.3 prepareBeanFactory(beanFactory)

BeanFactory的预准备工作 prepareBeanFactory(beanFactory):

prepareBeanFactory主要完成beanFactory的一些属性设置

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   // Tell the internal bean factory to use the context's class loader etc.
   beanFactory.setBeanClassLoader(getClassLoader());  //设置类加载器
   beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader())); //bean表达式解析器
   beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

   // Configure the bean factory with context callbacks.
   beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));  //添加一个BeanPostProcessor实现ApplicationContextAwareProcessor
//设置忽略的自动装配接口，表示这些接口的实现类不允许通过接口自动注入
   beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
   beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
   beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
   beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

   // BeanFactory interface not registered as resolvable type in a plain factory.
   // MessageSource registered (and found for autowiring) as a bean.
//注册可以自动装配的组件，就是可以在任何组件中允许自动注入的组件
   beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
   beanFactory.registerResolvableDependency(ResourceLoader.class, this);
   beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
   beanFactory.registerResolvableDependency(ApplicationContext.class, this);

   // Register early post-processor for detecting inner beans as ApplicationListeners.
   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

   //添加编译时的AspectJ
   if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
      beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
      // Set a temporary ClassLoader for type matching.
      beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   }

   // 给beanfactory容器中注册组件ConfigurableEnvironment、systemProperties、systemEnvironment
   if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
      beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
   }
   if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
      beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
   }
   if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
      beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
   }
}
```

#### 3.5 invokeBeanFactoryPostProcessors（beanFactory）

执行bean工厂的后置处理器

IOC容器初始化过程中有三个重要的步骤，

​          1：资源定位，2：bean定义的载入，3：将bean名称、bean定义以key-value形式注册到容器，这三个步骤都将在此完成。

 AbstractApplicationContext. invokeBeanFactoryPostProcessors方法实现：

```
调用链:

i1:org.springframework.context.support.AbstractApplicationContext#refresh

>i2:org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors

   >i3:org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors

     >i4:org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanDefinitionRegistryPostProcessors

       >i5:org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions

        > i6:org.springframework.context.annotation.ConfigurationClassParser#parse

           >i7:org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass

              >i8:org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass
```

i2:org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors

```java 
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, this.getBeanFactoryPostProcessors());
        if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean("loadTimeWeaver")) {
            beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
            beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
        }
    }
}
invokeBeanFactoryPostProcessors（beanFactory，getBeanFactoryPostProcessors()）方法内部执行实现了BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor这两个接口的Processor，先获取所有BeanDefinitionRegistryPostProcessor的实现，按优先级执行（是否实现PriorityOrdered优先级接口，是否实现Ordered顺序接口）；再以相同的策略执行所有BeanFactoryPostProcessor的实现。

----------------------------------------------------------------------
PostProcessorRegistrationDelegate. invokeBeanFactoryPostProcessors实现:

public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		Set<String> processedBeans = new HashSet<String>();
        
        //判断IOC 容器是不是BeanDefinitionRegistry的？
		if (beanFactory instanceof BeanDefinitionRegistry) {
		    //把IOC容器 强制转为BeanDefinitionRegistry类型的
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			//创建一个普通的PostProcessors的list的组件
			List<BeanFactoryPostProcessor> regularPostProcessors = new LinkedList<BeanFactoryPostProcessor>();
			//（重点）创建一个BeanDefinitionRegistryPostProcessor类型的list
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new LinkedList<BeanDefinitionRegistryPostProcessor>();
            
            //处理容器硬编码(new 出来的)带入的beanFacotryPostProcessors
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
			    //判断是不是BeanDefinitionRegistryPostProcessor
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
				    
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					//调用BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					//加入到list集合中
					registryProcessors.add(registryProcessor);
				}
				else {//判断不是BeanDefinitionRegistryPostProcessor
				    //加入到集合中
					regularPostProcessors.add(postProcessor);
				}
			}

            //创建一个当前注册的RegistryProcessors的集合
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<BeanDefinitionRegistryPostProcessor>();

			第一步:去容器中查询是否有BeanDefinitionRegistryPostProcessor类型的
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
			    //判断是不是实现了PriorityOrdered接口的
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				    //添加到currentRegistryProcessors的集合中
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					//添加到processedBeans的集合中
					processedBeans.add(ppName);
				}
			}
			//进行排序
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			//（重点）调用BeanDefinitionRegistryPostProcessors的postProcessBeanDefinitionRegistry方法
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			第二步:去容器中查询是否有BeanDefinitionRegistryPostProcessor类型的
			for (String ppName : postProcessorNames) {
			    //排除被处理过的，并且实现了Ordered接口的
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					//加到以处理的list中
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			//调用BeanDefinitionRegistryPostProcessors的postProcessBeanDefinitionRegistry方法
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			//调用普通的BeanDefinitionRegistryPostProcessors没用实现 PriorithOrdered和Ordered接口
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			//调用上诉实现了也实现了BeanFactoryPostProcessors的接口
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

        //去IOC 容器中获取BeanFactoryPostProcessor 类型的
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

        //分离实现了PriorityOrdered接口的 Ordered 接口的   普通的
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		List<String> orderedPostProcessorNames = new ArrayList<String>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
		
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		调用 PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		//调用 Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

	    //调用普通的
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanFactoryPostProcessor>();
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}
```



  ```java 
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
		List<BeanDefinitionHolder> configCandidates = new ArrayList<BeanDefinitionHolder>();
		//去IOC容器中的获取Bean定义的名称
		//	private volatile List<String> beanDefinitionNames = new ArrayList<String>(256);
        
        //没有解析之前，系统候选的bean定义配置(有自己的 有系统自带的)
		String[] candidateNames = registry.getBeanDefinitionNames();
        
        //循环Bean定义的名称 找出自己的传入的主配置类的bean定义信息  configCandidates
		for (String beanName : candidateNames) {
		    //去bean定义的map中获取对应的Bean定义对象
		    //	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>(256);
			BeanDefinition beanDef = registry.getBeanDefinition(beanName);
			//检查该bean定义对象是不是用来描述配置类
			if (ConfigurationClassUtils.isFullConfigurationClass(beanDef) ||
					ConfigurationClassUtils.isLiteConfigurationClass(beanDef)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
				}
			}
			else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
				configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
			}
		}

		// Return immediately if no @Configuration classes were found
		if (configCandidates.isEmpty()) {
			return;
		}

		//检查配置类排序
		Collections.sort(configCandidates, new Comparator<BeanDefinitionHolder>() {
			@Override
			public int compare(BeanDefinitionHolder bd1, BeanDefinitionHolder bd2) {
				int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
				int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
				return (i1 < i2) ? -1 : (i1 > i2) ? 1 : 0;
			}
		});

		// bean的名称生成策略
		SingletonBeanRegistry sbr = null;
		if (registry instanceof SingletonBeanRegistry) {
			sbr = (SingletonBeanRegistry) registry;
			if (!this.localBeanNameGeneratorSet && sbr.containsSingleton(CONFIGURATION_BEAN_NAME_GENERATOR)) {
				BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(CONFIGURATION_BEAN_NAME_GENERATOR);
				this.componentScanBeanNameGenerator = generator;
				this.importBeanNameGenerator = generator;
			}
		}

		/***创建一个配置类解析器
		1)元数据读取器工厂
		this.metadataReaderFactory = metadataReaderFactory;
		2)问题报告器
		this.problemReporter = problemReporter;
		//设置环境
		this.environment = environment;
		3)资源加载器
		this.resourceLoader = resourceLoader;
		4）创建了一个组件扫描器
		this.componentScanParser = new ComponentScanAnnotationParser(
				environment, resourceLoader, componentScanBeanNameGenerator, registry);
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, resourceLoader);
		****/
		
		ConfigurationClassParser parser = new ConfigurationClassParser(
				this.metadataReaderFactory, this.problemReporter, this.environment,
				this.resourceLoader, this.componentScanBeanNameGenerator, registry);
        
        
        //将要被解析的配置类(把自己的configCandidates加入到 候选的)
		Set<BeanDefinitionHolder> candidates = new LinkedHashSet<BeanDefinitionHolder>(configCandidates);
		//已经被解析的配置类(由于do while 那么mainclass就一定会被解析,被解析的size为1)
		Set<ConfigurationClass> alreadyParsed = new HashSet<ConfigurationClass>(configCandidates.size());
		do {
		    //通过配置解析器真正的解析配置类
			parser.parse(candidates);
			
			//进行校验
			parser.validate();
            
            //获取ConfigClass (把解析过的配置bean定义信息获取出来)
			Set<ConfigurationClass> configClasses = new LinkedHashSet<ConfigurationClass>(parser.getConfigurationClasses());
			configClasses.removeAll(alreadyParsed);

			// Read the model and create bean definitions based on its content
			if (this.reader == null) {
				this.reader = new ConfigurationClassBeanDefinitionReader(
						registry, this.sourceExtractor, this.resourceLoader, this.environment,
						this.importBeanNameGenerator, parser.getImportRegistry());
			}
			
			//@CompentScan是直接注册Bean定义信息的    但是通过获取@Import,@Bean这种的注解还没有注册的bean定义,
			this.reader.loadBeanDefinitions(configClasses);
			//把系统解析过我们自己的组件放在alreadyParsed
			alreadyParsed.addAll(configClasses);
            //清除解析过的 配置文件 
			candidates.clear();
			
			//已经注册的bean定义个数大于最新 开始系统+主配置类的(发生过解析)
			if (registry.getBeanDefinitionCount() > candidateNames.length) {
			    //获取系统+自己解析的+mainconfig的bean定义信息
				String[] newCandidateNames = registry.getBeanDefinitionNames();
				//系统的+mainconfig的bean定义信息
				Set<String> oldCandidateNames = new HashSet<String>(Arrays.asList(candidateNames));
				
				//已经解析过的自己的组件
				Set<String> alreadyParsedClasses = new HashSet<String>();
				for (ConfigurationClass configurationClass : alreadyParsed) {
					alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
				}
				
				for (String candidateName : newCandidateNames) {
				    //老的（系统+mainconfig） 不包含解析的
					if (!oldCandidateNames.contains(candidateName)) {
					    //把当前bean定义获取出来
						BeanDefinition bd = registry.getBeanDefinition(candidateName);
						//检查是否为解析过的
						if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
								!alreadyParsedClasses.contains(bd.getBeanClassName())) {
							//若不是解析过且通过检查的     把当前的bean定义 加入到candidates中	    
							candidates.add(new BeanDefinitionHolder(bd, candidateName));
						}
					}
				}
				// 把解析过的赋值给原来的 
				candidateNames = newCandidateNames;
			}
		}
		while (!candidates.isEmpty());  //还存主没有解析过的  再次解析

		// Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
		if (sbr != null) {
			if (!sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
				sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
			}
		}

		if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
			((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
		}
	}
  ```

 **i6源码解析 :org.springframework.context.annotation.ConfigurationClassParser#parse**

```java 
public void parse(Set<BeanDefinitionHolder> configCandidates) {
		this.deferredImportSelectors = new LinkedList<DeferredImportSelectorHolder>();

		for (BeanDefinitionHolder holder : configCandidates) {
			BeanDefinition bd = holder.getBeanDefinition();
			try {
			    //注解形式的bean定义信息
				if (bd instanceof AnnotatedBeanDefinition) {
				    //解析配置类的bean定义
					parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
				}
				else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
					parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
				}
				else {
					parse(bd.getBeanClassName(), holder.getBeanName());
				}
			}
			catch (BeanDefinitionStoreException ex) {
				throw ex;
			}
			catch (Throwable ex) {
				throw new BeanDefinitionStoreException(
						"Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
			}
		}

		processDeferredImportSelectors();
	}
	
----------------------------------------------------------------------
org.springframework.context.annotation.ConfigurationClassParser#parse
	org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass

protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
		if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
			return;
		}

		ConfigurationClass existingClass = this.configurationClasses.get(configClass);
		if (existingClass != null) {
			if (configClass.isImported()) {
				if (existingClass.isImported()) {
					existingClass.mergeImportedBy(configClass);
				}
				// Otherwise ignore new imported config class; existing non-imported class overrides it.
				return;
			}
			else {
				// Explicit bean definition found, probably replacing an import.
				// Let's remove the old one and go with the new one.
				this.configurationClasses.remove(configClass);
				for (Iterator<ConfigurationClass> it = this.knownSuperclasses.values().iterator(); it.hasNext();) {
					if (configClass.equals(it.next())) {
						it.remove();
					}
				}
			}
		}

		// 递归处理配置类及其超类层次结构。
		SourceClass sourceClass = asSourceClass(configClass);
		do {
			sourceClass = doProcessConfigurationClass(configClass, sourceClass);
		}
		while (sourceClass != null);

		this.configurationClasses.put(configClass, configClass);
	}
	
----------------------------------------------------------------------
org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass
	protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
			throws IOException {

		// Recursively process any member (nested) classes first
		processMemberClasses(configClass, sourceClass);

		//处理@PropertySource注解
		for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), PropertySources.class,
				org.springframework.context.annotation.PropertySource.class)) {
			if (this.environment instanceof ConfigurableEnvironment) {
				processPropertySource(propertySource);
			}
			else {
				logger.warn("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
						"]. Reason: Environment must implement ConfigurableEnvironment");
			}
		}

		//处理@ComponentScan注解
		
		//解析@ComponentScans注解的属性 封装成一个一个的componentscan对象
		Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
		if (!componentScans.isEmpty() &&!this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
			
			//循环componentScans的set
			for (AnnotationAttributes componentScan : componentScans) {
				// 立即执行扫描解析
				Set<BeanDefinitionHolder> scannedBeanDefinitions =this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
				//检查任何其他配置类的扫描定义集，并在需要时递归解析
				for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
				    //获取原始的bean定义信息
					BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
					if (bdCand == null) {
						bdCand = holder.getBeanDefinition();
					}
					//检查当前的bean定义信息是不是配置类  比如MainConfig的bean定义信息
					if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
					    //递归调用来解析MainConfig,解析出来配置类的中导入的bean定义信息
						parse(bdCand.getBeanClassName(), holder.getBeanName());
					}
				}
			}
		}

		//处理@Import注解   解析Import 注解的ImportSelector  ImportBeanDefinitionRegister,@Bean这种
		//存放在ConfigClass中
		processImports(configClass, sourceClass, getImports(sourceClass), true);

		//处理 @ImportResource annotations
		if (sourceClass.getMetadata().isAnnotated(ImportResource.class.getName())) {
			AnnotationAttributes importResource =
					AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
			String[] resources = importResource.getStringArray("locations");
			Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
			for (String resource : resources) {
				String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
				configClass.addImportedResource(resolvedResource, readerClass);
			}
		}

		// 处理  @Bean methods
		Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
		for (MethodMetadata methodMetadata : beanMethods) {
			configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
		}

		//处理接口
		processInterfaces(configClass, sourceClass);

		// 处理超类的
		if (sourceClass.getMetadata().hasSuperClass()) {
			String superclass = sourceClass.getMetadata().getSuperClassName();
			if (!superclass.startsWith("java") && !this.knownSuperclasses.containsKey(superclass)) {
				this.knownSuperclasses.put(superclass, configClass);
				// Superclass found, return its annotation metadata and recurse
				return sourceClass.getSuperClass();
			}
		}

		// No superclass -> processing is complete
		return null;
	}
	
----------------------------------------------------------------------
//通过组件扫描器进行真正的解析	
org.springframework.context.annotation.ComponentScanAnnotationParser#parse
Set<BeanDefinitionHolder>

	public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
		Assert.state(this.environment != null, "Environment must not be null");
		Assert.state(this.resourceLoader != null, "ResourceLoader must not be null");
        
        //创建一个类路径下的bean定义扫描器
		ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
				componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
        
        //为扫描器设置一个bean 名称的生成器
		Class<? extends BeanNameGenerator> generatorClass = componentScan.getClass("nameGenerator");
		boolean useInheritedGenerator = (BeanNameGenerator.class == generatorClass);
		scanner.setBeanNameGenerator(useInheritedGenerator ? this.beanNameGenerator :
				BeanUtils.instantiateClass(generatorClass));
        
        
		ScopedProxyMode scopedProxyMode = componentScan.getEnum("scopedProxy");
		if (scopedProxyMode != ScopedProxyMode.DEFAULT) {
			scanner.setScopedProxyMode(scopedProxyMode);
		}
		else {
			Class<? extends ScopeMetadataResolver> resolverClass = componentScan.getClass("scopeResolver");
			scanner.setScopeMetadataResolver(BeanUtils.instantiateClass(resolverClass));
		}

		scanner.setResourcePattern(componentScan.getString("resourcePattern"));

		for (AnnotationAttributes filter : componentScan.getAnnotationArray("includeFilters")) {
			for (TypeFilter typeFilter : typeFiltersFor(filter)) {
				scanner.addIncludeFilter(typeFilter);
			}
		}
		for (AnnotationAttributes filter : componentScan.getAnnotationArray("excludeFilters")) {
			for (TypeFilter typeFilter : typeFiltersFor(filter)) {
				scanner.addExcludeFilter(typeFilter);
			}
		}

		boolean lazyInit = componentScan.getBoolean("lazyInit");
		if (lazyInit) {
			scanner.getBeanDefinitionDefaults().setLazyInit(true);
		}

		Set<String> basePackages = new LinkedHashSet<String>();
		String[] basePackagesArray = componentScan.getStringArray("basePackages");
		for (String pkg : basePackagesArray) {
			String[] tokenized = StringUtils.tokenizeToStringArray(this.environment.resolvePlaceholders(pkg),
					ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
			basePackages.addAll(Arrays.asList(tokenized));
		}
		for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
			basePackages.add(ClassUtils.getPackageName(clazz));
		}
		if (basePackages.isEmpty()) {
			basePackages.add(ClassUtils.getPackageName(declaringClass));
		}

		scanner.addExcludeFilter(new AbstractTypeHierarchyTraversingFilter(false, false) {
			@Override
			protected boolean matchClassName(String className) {
				return declaringClass.equals(className);
			}
		});
		//真正扫描器扫描指定路径
		return scanner.doScan(StringUtils.toStringArray(basePackages));
	}

----------------------------------------------------------------------
//创建类路径下的bean定义扫描器
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
			Environment environment, ResourceLoader resourceLoader) {

		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;
        
        //使用默认的扫描规则
		if (useDefaultFilters) {
			registerDefaultFilters();
		}
		//设置环境变量
		setEnvironment(environment);
		//设置资源加载器
		setResourceLoader(resourceLoader);
	}
	
	//默认的扫描规则
	protected void registerDefaultFilters() {
	    //添加了Componet的解析，这就是我们为啥@Componet @Respository @Service @Controller的  @AspectJ
		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
		try {
		    //添加Jsr 250规范的注解
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
			logger.debug("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
		}
		try {
		    //JSR330的注解
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
			logger.debug("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-330 API not available - simply skip.
		}
	}
	----------------------------------------------------------------------
	使用扫描器去真正的扫描类,返回Set<BeanDefinitionHolder>
	org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan
	
	protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		//创建一个Bean定义 holder的 set
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
		//循环扫描路径
		for (String basePackage : basePackages) {
		    //找到候选的组件集合
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			//循环候选组件集合
			for (BeanDefinition candidate : candidates) {
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				candidate.setScope(scopeMetadata.getScopeName());
				//生成bean的名称
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
				//判断是不是抽象的beand定义
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
				//注解的bean定义西悉尼
				if (candidate instanceof AnnotatedBeanDefinition) {
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
				
				if (checkCandidate(beanName, candidate)) { //检查当前的和存主的bean定义是否有冲突
				    //把候选的组件封装成BeanDefinitionHolder
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					definitionHolder =AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					//加入到bean定义的集合中
					beanDefinitions.add(definitionHolder);
					//注册当前的bean定义信息
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}

    ----------------------------------------------------------------------
org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#findCandidateComponents 
    找到候选的组件 返回Set<BeanDefinition>的集合
	
	public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		//候选的bean定义信息
		Set<BeanDefinition> candidates = new LinkedHashSet<BeanDefinition>();
		try {
		    //拼接需要扫描包下面的类的路径   classpath*:com/tuling/testapplicationlistener/**/*.class
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + '/' + this.resourcePattern;
			//把路径解析成一个个.class文件		
			Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
			
			//循环.class文件的resource对象
			for (Resource resource : resources) {
				if (traceEnabled) {
					logger.trace("Scanning " + resource);
				}
				//判断class文件是否可读
				if (resource.isReadable()) {
					try {
					    //把resource对象 变为一个类的原信息读取器
						MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
						//判断类的源信息读取器是否为候选的组件
						if (isCandidateComponent(metadataReader)) {  //是候选的组件
						    //把类元信息读取器封装成一个ScannedGenericBeanDefinition
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							sbd.setResource(resource);
							sbd.setSource(resource);
							//是候选的组件
							if (isCandidateComponent(sbd)) {
								if (debugEnabled) {
									logger.debug("Identified candidate component class: " + resource);
								}
								//把当前解析出来的定义的加入到 BeanDefinition的集合中
								candidates.add(sbd);
							}
							else {
								if (debugEnabled) {
									logger.debug("Ignored because not a concrete top-level class: " + resource);
								}
							}
						}
						else {
							if (traceEnabled) {
								logger.trace("Ignored because not matching any filter: " + resource);
							}
						}
					}
					catch (Throwable ex) {
						throw new BeanDefinitionStoreException(
								"Failed to read candidate component class: " + resource, ex);
					}
				}
				else {
					if (traceEnabled) {
						logger.trace("Ignored because not readable: " + resource);
					}
				}
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
		}
		return candidates;
	}	
----------------------------------------------------------------------
是不是需要扫描的组件	
org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#isCandidateComponent
	protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
		//是不是被排除的
		for (TypeFilter tf : this.excludeFilters) {
			if (tf.match(metadataReader, this.metadataReaderFactory)) {
				return false;
			}
		}
		//在被包含的组件
		for (TypeFilter tf : this.includeFilters) {
			if (tf.match(metadataReader, this.metadataReaderFactory)) {
				return isConditionMatch(metadataReader);
			}
		}
		return false;
	}	
	
	是否能够进行@Conditional判断
   ---------------------------------------------------------------------- org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#isConditionMatch
	private boolean isConditionMatch(MetadataReader metadataReader) {
		if (this.conditionEvaluator == null) {
			this.conditionEvaluator = new ConditionEvaluator(getRegistry(), getEnvironment(), getResourceLoader());
		}
		return !this.conditionEvaluator.shouldSkip(metadataReader.getAnnotationMetadata());
	}	
	
```



### getBean()

```java
public Object getBean(String name) throws BeansException { 
    return doGetBean(name, null, null, false); 
}
```

#### doGetBean()

```java
protected <T> T doGetBean(String name, Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException {
    /** 
     * 转换对应的beanName 你们可能认为传入进来的name 不就是beanName么？ 
     * 传入进来的可能是别名,也有可能是factoryBean 
     * 1)去除factoryBean的修饰符 name="&instA"=====>instA 
     * 2)取指定的alias所表示的最终beanName 比如传入的是别名为ia---->指向为instA的bean，那么就返回instA 		 **/
    final String beanName = this.transformedBeanName(name);
	/** 
	* 设计的精髓 
	* 检查实例缓存中对象工厂缓存中是包含对象(从这里返回的可能是实例话好的,也有可能是没有实例化好的) 
	* 为什么要这段代码? 
	* 因为单实例bean创建可能存主依赖注入的情况，而为了解决循环依赖问题，在对象刚刚创建好(属性还没有赋值) 
	* 的时候，就会把对象包装为一个对象工厂暴露出去(加入到对象工厂缓存中),一但下一个bean要依赖他，就直接可以从缓存中获取. 
	* */ 
    //直接从缓存中获取或者从对象工厂缓存去取。
    Object sharedInstance = this.getSingleton(beanName);
    Object bean;
    if (sharedInstance != null && args == null) {
        if (this.logger.isDebugEnabled()) {
            if (this.isSingletonCurrentlyInCreation(beanName)) {
                this.logger.debug("Returning eagerly cached instance of singleton bean '" + beanName + "' that is not fully initialized yet - a consequence of a circular reference");
            } else {
                this.logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }
		/** 
		* 若从缓存中的sharedInstance是原始的bean(属性还没有进行实例化,那么在这里进行处理) 
		* 或者是factoryBean 返回的是工厂bean的而不是我们想要的getObject()返回的bean ,就会在这里处理 
		**/
        bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
    } else {
        if (this.isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }
		// 获取父类容器
        BeanFactory parentBeanFactory = this.getParentBeanFactory();
        //如果beanDefinitionMap中所有以及加载的bean不包含 本次加载的beanName，那么尝试取父容器取检测
        if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
            String nameToLookup = this.originalBeanName(name);
            if (args != null) {
                //父容器递归查询
                return parentBeanFactory.getBean(nameToLookup, args);
            }

            return parentBeanFactory.getBean(nameToLookup, requiredType);
        }
		//如果这里不是做类型检查，而是创建bean,这里需要标记一下
        if (!typeCheckOnly) {
            this.markBeanAsCreated(beanName);
        }

        try {
            /** 
            * 合并父 BeanDefinition 与子 BeanDefinition，后面会单独分析这个方法 
            **/
            final RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
            this.checkMergedBeanDefinition(mbd, beanName, args);
            //用来处理bean加载的顺序依赖 比如要创建instA 的情况下 必须需要先创建instB 
            /** 
            * <bean id="beanA" class="BeanA" depends-on="beanB">
            * <bean id="beanB" class="BeanB" depends-on="beanA"> 
            * 创建A之前 需要创建B 创建B之前需要创建A 就会抛出异常 
            **/
            String[] dependsOn = mbd.getDependsOn();
            String[] var11;
            if (dependsOn != null) {
                var11 = dependsOn;
                int var12 = dependsOn.length;

                for(int var13 = 0; var13 < var12; ++var13) {
                    String dep = var11[var13];
                    if (this.isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }
                    //注册依赖
                    this.registerDependentBean(dep, beanName);

                    try {
                        //优先创建依赖的对象
                        this.getBean(dep);
                    } catch (NoSuchBeanDefinitionException var24) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "'" + beanName + "' depends on missing bean '" + dep + "'", var24);
                    }
                }
            }
			//创建bean（单例的 ）
            if (mbd.isSingleton()) {
                //创建单实例bean
                sharedInstance = this.getSingleton(beanName, new ObjectFactory<Object>() {
                    //在getSingleton房中进行回调用的
                    public Object getObject() throws BeansException {
                        try {
                            return AbstractBeanFactory.this.createBean(beanName, mbd, args);
                        } catch (BeansException var2) {
                            AbstractBeanFactory.this.destroySingleton(beanName);
                            throw var2;
                        }
                    }
                });
                bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            //创建非单实例bean
            } else if (mbd.isPrototype()) {
                var11 = null;

                Object prototypeInstance;
                try {
                    this.beforePrototypeCreation(beanName);
                    prototypeInstance = this.createBean(beanName, mbd, args);
                } finally {
                    this.afterPrototypeCreation(beanName);
                }

                bean = this.getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            } else {
                String scopeName = mbd.getScope();
                Scope scope = (Scope)this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }

                try {
                    Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                        public Object getObject() throws BeansException {
                            AbstractBeanFactory.this.beforePrototypeCreation(beanName);

                            Object var1;
                            try {
                                var1 = AbstractBeanFactory.this.createBean(beanName, mbd, args);
                            } finally {
                                AbstractBeanFactory.this.afterPrototypeCreation(beanName);
                            }

                            return var1;
                        }
                    });
                    bean = this.getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                } catch (IllegalStateException var23) {
                    throw new BeanCreationException(beanName, "Scope '" + scopeName + "' is not active for the current thread; consider defining a scoped proxy for this bean if you intend to refer to it from a singleton", var23);
                }
            }
        } catch (BeansException var26) {
            this.cleanupAfterBeanCreationFailure(beanName);
            throw var26;
        }
    }

    if (requiredType != null && bean != null && !requiredType.isInstance(bean)) {
        try {
            return this.getTypeConverter().convertIfNecessary(bean, requiredType);
        } catch (TypeMismatchException var25) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Failed to convert bean '" + name + "' to required type '" + ClassUtils.getQualifiedName(requiredType) + "'", var25);
            }

            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
        }
    } else {
        return bean;
    }
}


```

#### getSingleton()

**去缓存中获取**bean**源码分析** 

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //去缓存map中获取以及实例化好的bean对象
    Object singletonObject = this.singletonObjects.get(beanName);
    //缓存中没有获取到,并且当前bean是否在正在创建
    if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
        //加锁，防止并发创建
        synchronized(this.singletonObjects) {
            // 保存早期对象缓存中是否有该对象
            singletonObject = this.earlySingletonObjects.get(beanName);
            // 早期对象缓存没有
            if (singletonObject == null && allowEarlyReference) {
                // 早期对象暴露工厂缓存（用来解决循环依赖的）
                ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // 调用方法获取早期对象
                    singletonObject = singletonFactory.getObject();
                    // 放入到早期对象缓存中
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }

    return singletonObject != NULL_OBJECT ? singletonObject : null;
}

```

#### getObjectForBeanInstance（）

**在**Bean**的生命周期中，**getObjectForBeanInstance**方法是频繁使用的方法，无论是从缓存中获取出来的**bean**还是** 

**根据**scope**创建出来的**bean,**都要通过该方法进行检查。** 

**①**:**检查当前**bean**是否为**factoryBean,**如果是就需要调用该对象的**getObject()**方法来返回我们需要的**bean**对象** 

```java
protected Object getObjectForBeanInstance(Object beanInstance, String name, String beanName, RootBeanDefinition mbd) {
    //判断name为以 &开头的但是 又不是factoryBean类型的 就抛出异常
    if (BeanFactoryUtils.isFactoryDereference(name) && !(beanInstance instanceof FactoryBean)) {
        throw new BeanIsNotAFactoryException(this.transformedBeanName(name), beanInstance.getClass());
        /** 
        * 现在我们有了这个bean，它可能是一个普通bean 也有可能是工厂bean 
        * 1)若是工厂bean，我们使用他来创建实例，当如果想要获取的是工厂实例而不是工厂bean的getObject()对应的bean,我们应该传入&开头 
        **/
    } else if (beanInstance instanceof FactoryBean && !BeanFactoryUtils.isFactoryDereference(name)) {
        //加载factoryBean
        Object object = null;
        if (mbd == null) {
            /**
            * 如果 mbd 为空，则从缓存中加载 bean。FactoryBean 生成的单例 bean 会被缓存 
            * 在 factoryBeanObjectCache 集合中，不用每次都创建 
            */
            object = this.getCachedObjectForFactoryBean(beanName);
        }

        if (object == null) {
            // 经过前面的判断，到这里可以保证 beanInstance 是 FactoryBean 类型的，所以可以进行类型转换
            FactoryBean<?> factory = (FactoryBean)beanInstance;
            // 如果 mbd 为空，则判断是否存在名字为 beanName 的 BeanDefinition
            if (mbd == null && this.containsBeanDefinition(beanName)) {
                //合并我们的bean定义
                mbd = this.getMergedLocalBeanDefinition(beanName);
            }

            boolean synthetic = mbd != null && mbd.isSynthetic();
            // 调用 getObjectFromFactoryBean 方法继续获取实例
            object = this.getObjectFromFactoryBean(factory, beanName, !synthetic);
        }

        return object;
    } else {
        return beanInstance;
    }
}

-------------------------------------核心方法-----------------------------------------------
    
 protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
    /** 
    * FactoryBean 也有单例和非单例之分，针对不同类型的 FactoryBean，这里有两种处理方式： 
    * 1. 单例 FactoryBean 生成的 bean 实例也认为是单例类型。需放入缓存中，供后续重复使用 
    * 2. 非单例 FactoryBean 生成的 bean 实例则不会被放入缓存中，每次都会创建新的实例 
    */
    if (factory.isSingleton() && this.containsSingleton(beanName)) {
        //加锁，防止重复创建 可以使用缓存提高性能
        synchronized(this.getSingletonMutex()) {
            // 从缓存中获取
            Object object = this.factoryBeanObjectCache.get(beanName);
            if (object == null) {
                // 没有获取到，使用factoryBean的getObject（）方法去获取对象
                object = this.doGetObjectFromFactoryBean(factory, beanName);
                Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
                if (alreadyThere != null) {
                    object = alreadyThere;
                } else {
                    if (object != null && shouldPostProcess) {
                        if (this.isSingletonCurrentlyInCreation(beanName)) {
                            return object;
                        }

                        this.beforeSingletonCreation(beanName);

                        try {
                            //调用ObjectFactory的后置处理器
                            object = this.postProcessObjectFromFactoryBean(object, beanName);
                        } catch (Throwable var14) {
                            throw new BeanCreationException(beanName, "Post-processing of FactoryBean's singleton object failed", var14);
                        } finally {
                            this.afterSingletonCreation(beanName);
                        }
                    }

                    if (this.containsSingleton(beanName)) {
                        this.factoryBeanObjectCache.put(beanName, object != null ? object : NULL_OBJECT);
                    }
                }
            }

            return object != NULL_OBJECT ? object : null;
        }
    } else {
        Object object = this.doGetObjectFromFactoryBean(factory, beanName);
        if (object != null && shouldPostProcess) {
            try {
                object = this.postProcessObjectFromFactoryBean(object, beanName);
            } catch (Throwable var17) {
                throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", var17);
            }
        }

        return object;
    }
}
-----------------------------------------------------------------------------------------
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, String beanName) throws BeanCreationException {
    Object object;
    try {
        // 安全检查
        if (System.getSecurityManager() != null) {
            AccessControlContext acc = this.getAccessControlContext();

            try {
                object = AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
                    public Object run() throws Exception {
                        //调用工厂bean的getObject()方法
                        return factory.getObject();
                    }
                }, acc);
            } catch (PrivilegedActionException var6) {
                throw var6.getException();
            }
        } else {
            //调用工厂bean的getObject()方法
            object = factory.getObject();
        }
    } catch (FactoryBeanNotInitializedException var7) {
        throw new BeanCurrentlyInCreationException(beanName, var7.toString());
    } catch (Throwable var8) {
        throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", var8);
    }

    if (object == null && this.isSingletonCurrentlyInCreation(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName, "FactoryBean which is currently in creation returned null from getObject");
    } else {
        return object;
    }
}

-------------------------------------postProcessObjectFromFactoryBean()------------------------------
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
    Object result = existingBean;
    Iterator var4 = this.getBeanPostProcessors().iterator();

    do {
        if (!var4.hasNext()) {
            return result;
        }

        BeanPostProcessor processor = (BeanPostProcessor)var4.next();
        result = processor.postProcessAfterInitialization(result, beanName);
    } while(result != null);

    return result;
}
```

#### getMergedLocalBeanDefinition

**将** bean**定义转为**RootBeanDifination ,**合并父子**bean**定义**

```java
<bean id="tulingParentCompent" class="com.tuling.testparentsonbean.TulingParentCompent" abstract="true">
	<property name="tulingCompent" ref="tulingCompent"></property> 
</bean> 
<bean id="tulingSonCompent" class="com.tuling.testparentsonbean.TulingSonCompent" parent="tulingParentCompent"></bean> 
<bean id="tulingLog" class="com.tuling.testcreatebeaninst.TulingLog"></bean> 
protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException { 
//去合并的bean定义缓存中 判断当前的bean是否合并过 RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName); if (mbd != null) { return mbd; }
//没有合并，调用合并分方法 return getMergedBeanDefinition(beanName, getBeanDefinition(beanName)); }

// beanName获取到当前的bean定义信息
public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
        BeanDefinition bd = (BeanDefinition)this.beanDefinitionMap.get(beanName);
        if (bd == null) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No bean named '" + beanName + "' found in " + this);
            }

            throw new NoSuchBeanDefinitionException(beanName);
        } else {
            return bd;
        }
    }
------------------------------------------------------------------------------------
protected RootBeanDefinition getMergedBeanDefinition(String beanName, BeanDefinition bd) throws BeanDefinitionStoreException {
	return this.getMergedBeanDefinition(beanName, bd, (BeanDefinition)null);
}

protected RootBeanDefinition getMergedBeanDefinition(String beanName, BeanDefinition bd, BeanDefinition containingBd) throws BeanDefinitionStoreException {
        synchronized(this.mergedBeanDefinitions) {
            RootBeanDefinition mbd = null;
            //去缓存中获取一次bean定义
            if (containingBd == null) {
                mbd = (RootBeanDefinition)this.mergedBeanDefinitions.get(beanName);
            }

		   //尝试没有获取到
            if (mbd == null) {
            	//当前bean定义是否有父bean
                if (bd.getParentName() == null) {
                // 转为rootBeanDefinaition 然后深度克隆返回
                    if (bd instanceof RootBeanDefinition) {
                        mbd = ((RootBeanDefinition)bd).cloneBeanDefinition();
                    } else {
                        mbd = new RootBeanDefinition(bd);
                    }
                } else {
                	//有父bean
               		//定义一个父的bean定义
                    BeanDefinition pbd;
                    try {
                    	//获取父bena的名称
                        String parentBeanName = this.transformedBeanName(bd.getParentName());
                        /** 
                        * 判断父类 beanName 与子类 beanName 名称是否相同。若相同，则父类 bean 一定 
                        * 在父容器中。原因也很简单，容器底层是用 Map 缓存 <beanName, bean> 键值对 
                        * 的。同一个容器下，使用同一个 beanName 映射两个 bean 实例显然是不合适的。 
                        * 有的朋友可能会觉得可以这样存储：<beanName, [bean1, bean2]> ，似乎解决了 
                        * 一对多的问题。但是也有问题，调用 getName(beanName) 时，到底返回哪个 bean 
                        * 实例好呢？ 
                        */
                        if (!beanName.equals(parentBeanName)) {
                            pbd = this.getMergedBeanDefinition(parentBeanName);
                        } else {
                        	//获取父容器
                            BeanFactory parent = this.getParentBeanFactory();
                            if (!(parent instanceof ConfigurableBeanFactory)) {
                                throw new NoSuchBeanDefinitionException(parentBeanName, "Parent name '" + parentBeanName + "' is equal to bean name '" + beanName + "': cannot be resolved without an AbstractBeanFactory parent");
                            }
						 //从父容器获取父bean的定义 //若父bean中有父bean 存储递归合并
                            pbd = ((ConfigurableBeanFactory)parent).getMergedBeanDefinition(parentBeanName);
                        }
                    } catch (NoSuchBeanDefinitionException var10) {
                        throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName, "Could not resolve parent bean definition '" + bd.getParentName() + "'", var10);
                    }
				 //以父 BeanDefinition 的配置信息为蓝本创建 RootBeanDefinition，也就是“已合并的 BeanDefinition”
                    mbd = new RootBeanDefinition(pbd);
                    //用子 BeanDefinition 中的属性覆盖父 BeanDefinition 中的属性
                    mbd.overrideFrom(bd);
                }

			  //若之前没有定义,就把当前的设置为单例的
                if (!StringUtils.hasLength(mbd.getScope())) {
                    mbd.setScope("singleton");
                }

                if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {
                    mbd.setScope(containingBd.getScope());
                }
			  // 缓存合并后的 BeanDefinition
                if (containingBd == null && this.isCacheBeanMetadata()) {
                    this.mergedBeanDefinitions.put(beanName, mbd);
                }
            }

            return mbd;
        }
    }
```

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton**根据**scope **的添加来创建**bean

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "'beanName' must not be null");
    synchronized(this.singletonObjects) {
        // 从缓存中获取对象
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName, "Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }

            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
            }

            //打标.....把正在创建的bean 的标识设置为ture singletonsCurrentlyInDestruction
            this.beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = this.suppressedExceptions == null;
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet();
            }

            try {
                //调用单实例bean的创建
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            } catch (IllegalStateException var16) {
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw var16;
                }
            } catch (BeanCreationException var17) {
                BeanCreationException ex = var17;
                if (recordSuppressedExceptions) {
                    Iterator var8 = this.suppressedExceptions.iterator();

                    while(var8.hasNext()) {
                        Exception suppressedException = (Exception)var8.next();
                        ex.addRelatedCause(suppressedException);
                    }
                }

                throw ex;
            } finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }

                this.afterSingletonCreation(beanName);
            }

            if (newSingleton) {
                //加载到缓存中
                this.addSingleton(beanName, singletonObject);
            }
        }

        return singletonObject != NULL_OBJECT ? singletonObject : null;
    }
}
---------------------this.addSingleton(beanName, singletonObject);-------------------
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized(this.singletonObjects) {
        // 加入缓存
        this.singletonObjects.put(beanName, singletonObject != null ? singletonObject : NULL_OBJECT);
        // 从早期对象缓存和解决缓存依赖中移除
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```

#### createBean（）

```JAVA
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Creating instance of bean '" + beanName + "'");
    }

    RootBeanDefinition mbdToUse = mbd;
    //根据bean定义和beanName解析class
    Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    try {
        mbdToUse.prepareMethodOverrides();
    } catch (BeanDefinitionValidationException var7) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var7);
    }

    Object beanInstance;
    try {
        // 给bean的后置处理器一个机会来生成一个代理对象返回，在aop 模板进行详细讲解
        beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
        if (beanInstance != null) {
            return beanInstance;
        }
    } catch (Throwable var8) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var8);
    }

    // 真正进行主要的业务逻辑方法来进行创建bean
    beanInstance = this.doCreateBean(beanName, mbdToUse, args);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Finished creating instance of bean '" + beanName + "'");
    }

    return beanInstance;
}
----------------------beanInstance = this.doCreateBean(beanName, mbdToUse, args);--------------------
// 正真的创建bean的逻辑
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
    }

    // 调用构造方法创建bean的实例（）
    if (instanceWrapper == null) {
        
        instanceWrapper = this.createBeanInstance(beanName, mbd, args);
    }

    /**
    * 如果纯在工厂方法则使用工厂方法进行初始化
    * 一个类有多个构造方法，每个构造函数都有不同的参数，所有需要根据参数锁定构造函数进行初始化
    * 如果既不存在工厂方法也不存在带有参数的构造函数，则使用默认的构造函数进行bean 的实例化
    **/
    final Object bean = instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null;
    Class<?> beanType = instanceWrapper != null ? instanceWrapper.getWrappedClass() : null;
    mbd.resolvedTargetType = beanType;
    synchronized(mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                /**
                * bean 的后置处理器
                * bean 合并后的处理， Autowired注解正是通过此方法实现诸如此类的预解析。
                */
                this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            } catch (Throwable var17) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Post-processing of merged bean definition failed", var17);
            }

            mbd.postProcessed = true;
        }
    }

    // 判断当前bean是否需要暴露在缓存对象中
    boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
    if (earlySingletonExposure) {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
        }
	 	// 暴露早期对象的缓存中用于解决依赖的
        this.
            addSingletonFactory(beanName, new ObjectFactory<Object>() {
            public Object getObject() throws BeansException {
                return AbstractAutowireCapableBeanFactory.this.getEarlyBeanReference(beanName, mbd, bean);
            }
        });
    }

    Object exposedObject = bean;

    try {
        // 为当前的bean填充属性，发现依赖等。。。解决循环依赖就是在这个地方
        this.populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            exposedObject = this.initializeBean(beanName, exposedObject, mbd);
        }
    } catch (Throwable var18) {
        if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
            throw (BeanCreationException)var18;
        }

        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var18);
    }

    if (earlySingletonExposure) {
         // 此时一级缓存肯定还没数据，但是呢此时候二级缓存earlySingletonObjects也没数据
	    //注意，注意：第二参数为false  表示不会再去三级缓存里查了~~~
		// 此处非常巧妙的一点：因为上面各式各样的实例化、初始化的后置处理器都执行了，如果你在上面执行了这一句
		//  ((ConfigurableListableBeanFactory)this.beanFactory).registerSingleton(beanName, bean);
		// 那么此处得到的earlySingletonReference 的引用最终会是你手动放进去的Bean最终返回，完美的实现了"偷天换日" 特别适合中间件的设计
		// 我们知道，执行完此doCreateBean后执行addSingleton()  其实就是把自己再添加一次  **再一次强调，完美实现偷天换日**
        // 从缓存中获取对象，只有bean没有循环依赖 earlySingletonReference 才会为空
        Object earlySingletonReference = this.getSingleton(beanName, false);
        if (earlySingletonReference != null) {
            // 这个意思是如果经过了initializeBean()后，exposedObject还是木有变，那就可以大胆放心的返回了
		   // initializeBean会调用后置处理器，这个时候可以生成一个代理对象，那这个时候它哥俩就不会相等了 走else去判断吧
            // 检查当前bean在初始化方法中有没有被增强过（代理过）
            if (exposedObject == bean) {
                exposedObject = earlySingletonReference;
            } else if (!this.allowRawInjectionDespiteWrapping && this.hasDependentBean(beanName)) {
                String[] dependentBeans = this.getDependentBeans(beanName);
                Set<String> actualDependentBeans = new LinkedHashSet(dependentBeans.length);
                String[] var12 = dependentBeans;
                int var13 = dependentBeans.length;

                for(int var14 = 0; var14 < var13; ++var14) {
                    String dependentBean = var12[var14];
                    if (!this.removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                        actualDependentBeans.add(dependentBean);
                    }
                }

                if (!actualDependentBeans.isEmpty()) {
                    throw new BeanCurrentlyInCreationException(beanName, "Bean with name '" + beanName + "' has been injected into other beans [" + StringUtils.collectionToCommaDelimitedString(actualDependentBeans) + "] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
                }
            }
        }
    }

    try {
        // 注册DisposableBean。如果配置了destroy-method，这里需要注册以便在注销时候调用。
        this.registerDisposableBeanIfNecessary(beanName, bean, mbd);
        return exposedObject;
    } catch (BeanDefinitionValidationException var16) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", var16);
    }
}
------------------instanceWrapper = this.createBeanInstance(beanName, mbd, args);--------------------
因为一个类可能有多个构造函数，所以需要根据配置文件中配置的参数或者传入的参数确定最终调用的构造函数。因为判断过程会比较消耗性能， 所以Spring会将解析、确定好的构造函数缓存到BeanDefinition中的resolvedConstructorOrFactoryMethod字段中。在下次创建相同bean的时候(多例)， 会直接从RootBeanDefinition中的属性resolvedConstructorOrFactoryMethod缓存的值获取，避免再次解析
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
        Class<?> beanClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
        if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
        // 工厂方法不为空则使工厂方法初始化策略 也就是bean的配置过程中设置了factory-method方法
        } else if (mbd.getFactoryMethodName() != null) {
            return this.instantiateUsingFactoryMethod(beanName, mbd, args);
        } else {
            boolean resolved = false;
            boolean autowireNecessary = false;
            if (args == null) {
            // 如果已缓存的解析的构造函数或者工厂方法不为空，则可以利用构造函数解析 
            
            // 因为需要根据参数确认到底使用哪个构造函数，该过程比较消耗性能，所有采用缓存机制（缓存到bean定义中）
                synchronized(mbd.constructorArgumentLock) {
                    if (mbd.resolvedConstructorOrFactoryMethod != null) {
                        resolved = true;
                        //从bean定义中解析出对应的构造函数
                        autowireNecessary = mbd.constructorArgumentsResolved;
                    }
                }
            }
			//已经解析好了，直接注入即可
            if (resolved) {
            	//autowire 自动注入，调用构造函数自动注入
                return autowireNecessary ? this.autowireConstructor(beanName, mbd, (Constructor[])null, (Object[])null) : this.instantiateBean(beanName, mbd);
            } else {
            //使用默认的构造函数
                Constructor<?>[] ctors = this.determineConstructorsFromBeanPostProcessors(beanClass, beanName);
                //根据beanClass和beanName去bean的后置处理器中获取构造方法（SmartInstantiationAwareBeanPostProcessor 
                return ctors == null && mbd.getResolvedAutowireMode() != 3 && !mbd.hasConstructorArgumentValues() && ObjectUtils.isEmpty(args) ? this.instantiateBean(beanName, mbd) : this.autowireConstructor(beanName, mbd, ctors, args);
            }
        }
    }
--------------------提前暴露对象---------------------
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized(this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }

    }
}
/** 
* 在配置文件中是我们使用的@Bean的形式都是通过工厂方法的形式来实例化对象 
*
*/
=======autowireConstructor(beanName, mbd, ctors, args);很复杂 很复杂 =========================== 
    1、autowireConstructor（带参）
对于实例的创建，Spring分为通用的实例化（默认无参构造函数），以及带有参数的实例化 下面代码是带有参数情况的实例化。因为需要确定使用的构造函数，所以需要有大量工作花在根据参数个数、类型来确定构造函数上（因为带参的构造函数可能有很多个 ）： 
public BeanWrapper autowireConstructor(final String beanName, final RootBeanDefinition mbd, Constructor<?>[] chosenCtors, Object[] explicitArgs) {
		//用于包装bean实例的
        BeanWrapperImpl bw = new BeanWrapperImpl();
        this.beanFactory.initBeanWrapper(bw);
        //定义一个参数用来保存 使用的构造函数
        final Constructor<?> constructorToUse = null;
        //定义一个参数用来保存 使用的参数持有器
        ConstructorResolver.ArgumentsHolder argsHolderToUse = null;
        //用于保存 使用的参数
        final Object[] argsToUse = null;
        //判断传入的参数是不是空
        if (explicitArgs != null) {
        //赋值给argsToUse 然后执行使用
            argsToUse = explicitArgs;
        //传入进来的是空,需要从配置文件中解析出来
        } else {
        	//存解析的参数
            Object[] argsToResolve = null;
            //加锁
            synchronized(mbd.constructorArgumentLock) {
           		//从缓存中获取解析出来的构造参数
                constructorToUse = (Constructor)mbd.resolvedConstructorOrFactoryMethod;
                //缓存中有构造方法
                if (constructorToUse != null && mbd.constructorArgumentsResolved) {
                	//从缓存中获取解析的参数
                    argsToUse = mbd.resolvedConstructorArguments;
                    if (argsToUse == null) {
                    	//没有缓存的参数，就需要获取配置文件中配置的参数
                        argsToResolve = mbd.preparedConstructorArguments;
                    }
                }
            }
		   //缓存中有构造器参数
            if (argsToResolve != null) {
            //解析参数类型， 如给定方法的构造函数 A( int , int ) 则通过此方法后就会把配置中的 
            //（ ”1”，”l”）转换为 (1 , 1) 
                argsToUse = this.resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve);
            }
        }

		//如果没有缓存，就需要从构造函数开始解析
        if (constructorToUse == null) {
        	//是否需要解析构造函数
            boolean autowiring = chosenCtors != null || mbd.getResolvedAutowireMode() == 3;
            ConstructorArgumentValues resolvedValues = null;
            //用来保存getBeans传入进来的参数的个数
            int minNrOfArgs;
            if (explicitArgs != null) {
                minNrOfArgs = explicitArgs.length;
            } else {
           		//从bean定义中解析出来构造参数的对象
                ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
                resolvedValues = new ConstructorArgumentValues();
                //计算出构造参数参数的个数
                minNrOfArgs = this.resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
            }

            //如果传入的构造器数组不为空，就使用传入的构造器参数，否则通过反射获取class中定义的构造器 
            Constructor<?>[] candidates = chosenCtors;
            //传入的构造参数为空
            if (chosenCtors == null) {
            //解析出对应的bean的class
                Class beanClass = mbd.getBeanClass();

                try {
                	//获取构造方法
                    candidates = mbd.isNonPublicAccessAllowed() ? beanClass.getDeclaredConstructors() : beanClass.getConstructors();
                } catch (Throwable var25) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Resolution of declared constructors on bean Class [" + beanClass.getName() + "] from ClassLoader [" + beanClass.getClassLoader() + "] failed", var25);
                }
            }

			//给构造函数排序，public构造函数优先、参数数量降序排序
            AutowireUtils.sortConstructors(candidates);
            int minTypeDiffWeight = 2147483647;
            //不确定的构造函数
            Set<Constructor<?>> ambiguousConstructors = null;
            LinkedList<UnsatisfiedDependencyException> causes = null;
            Constructor[] var16 = candidates;
            int var17 = candidates.length;

            //根据从bean定义解析出来的参数个数来推算出构造函数 
            //循环所有的构造函数 查找合适的构造函数
            for(int var18 = 0; var18 < var17; ++var18) {
           		//获取正在循环的构造函数的个数
                Constructor<?> candidate = var16[var18];
                Class<?>[] paramTypes = candidate.getParameterTypes();
                if (constructorToUse != null && argsToUse.length > paramTypes.length) {
                //如果找到了已经构造函数,并且已经确定的构造函数的参数个数>正在当前循环的 那么就直接返回(candidates参数个数已经是排序的 ) 
                    break;
                }
			   
                if (paramTypes.length >= minNrOfArgs) {
                    ConstructorResolver.ArgumentsHolder argsHolder;
                    //从bean定义中解析的构造函数的参数对象
                    if (resolvedValues != null) {
                        try {
                        //从注解 @ConstructorProperties获取参数名称
                            String[] paramNames = ConstructorResolver.ConstructorPropertiesChecker.evaluate(candidate, paramTypes.length);
                            //没有获取到
                            if (paramNames == null) {
                            //去容器中获取一个参数探测器
                                ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
                                if (pnd != null) {
                                //通过参数探测器去探测当前正在循环的构造参数
                                    paramNames = pnd.getParameterNames(candidate);
                                }
                            }
						  // 根据参数名称和数据类型创建参数持有器
                            argsHolder = this.createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames, this.getUserDeclaredConstructor(candidate), autowiring);
                        } catch (UnsatisfiedDependencyException var26) {
                            if (this.beanFactory.logger.isTraceEnabled()) {
                                this.beanFactory.logger.trace("Ignoring constructor [" + candidate + "] of bean '" + beanName + "': " + var26);
                            }

                            if (causes == null) {
                                causes = new LinkedList();
                            }

                            causes.add(var26);
                            continue;
                        }
                    } else {
                    	//解析出来的参数个数和从外面传递进来的个数不相等 进入下一个循环
                        if (paramTypes.length != explicitArgs.length) {
                            continue;
                        }
					  //把外面传递进来的参数封装为一个参数持有器
                        argsHolder = new ConstructorResolver.ArgumentsHolder(explicitArgs);
                    }

				  // 探测是否有不确定性的构造函数存在， 例如不同构造函数的参数为父子关系
                    int typeDiffWeight = mbd.isLenientConstructorResolution() ? argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes);
                    //因为不同构造函数的参数个数相同，而且参数类型为父子关系，所以需要找出类型最符合的一个构造函数 
                    //Spring用一种权重的形式来表示类型差异程度，差异权重越小越优先
                    if (typeDiffWeight < minTypeDiffWeight) {
                    	//当前构造函数最为匹配的话，清空先前ambiguousConstructors列表
                        constructorToUse = candidate;
                        argsHolderToUse = argsHolder;
                        argsToUse = argsHolder.arguments;
                        minTypeDiffWeight = typeDiffWeight;
                        ambiguousConstructors = null;
                        //存在相同权重的构造器，将构造器添加到一个ambiguousConstructors列表变量中 
                        //注意,这时候constructorToUse 指向的仍是第一个匹配的构造函数
                    } else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
                        if (ambiguousConstructors == null) {
                            ambiguousConstructors = new LinkedHashSet();
                            ambiguousConstructors.add(constructorToUse);
                        }

                        ambiguousConstructors.add(candidate);
                    }
                }
            }
		   //还是没有找到构造函数 就抛出异常
            if (constructorToUse == null) {
                if (causes == null) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Could not resolve matching constructor (hint: specify index/type/name arguments for simple parameters to avoid type ambiguities)");
                }

                UnsatisfiedDependencyException ex = (UnsatisfiedDependencyException)causes.removeLast();
                Iterator var34 = causes.iterator();

                while(var34.hasNext()) {
                    Exception cause = (Exception)var34.next();
                    this.beanFactory.onSuppressedException(cause);
                }

                throw ex;
            }

            if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Ambiguous constructor matches found in bean '" + beanName + "' (hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " + ambiguousConstructors);
            }
			//把解析出来的构造函数加入到缓存中
            if (explicitArgs == null) {
                argsHolderToUse.storeCache(mbd, constructorToUse);
            }
        }
		//调用构造函数进行反射创建
        try {
            Object beanInstance;
            if (System.getSecurityManager() != null) {
                beanInstance = AccessController.doPrivileged(new PrivilegedAction<Object>() {
                    public Object run() {
                        return ConstructorResolver.this.beanFactory.getInstantiationStrategy().instantiate(mbd, beanName, ConstructorResolver.this.beanFactory, constructorToUse, argsToUse);
                    }
                }, this.beanFactory.getAccessControlContext());
            } else {
            	//获取生成实例策略类调用实例方法
                beanInstance = this.beanFactory.getInstantiationStrategy().instantiate(mbd, beanName, this.beanFactory, constructorToUse, argsToUse);
            }

            bw.setBeanInstance(beanInstance);
            return bw;
        } catch (Throwable var24) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Bean instantiation via constructor failed", var24);
        }
    }
-----------------------------instantiate ----------------------------------
public Object instantiate(RootBeanDefinition bd, String beanName, BeanFactory owner, final Constructor<?> ctor, Object... args) {
        if (bd.getMethodOverrides().isEmpty()) {
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged(new PrivilegedAction<Object>() {
                    public Object run() {
                        ReflectionUtils.makeAccessible(ctor);
                        return null;
                    }
                });
            }
			//调用反射创建
            return BeanUtils.instantiateClass(ctor, args);
        } else {
            return this.instantiateWithMethodInjection(bd, beanName, owner, ctor, args);
        }
    }

```

#### addSingletonFactory()

**暴露对象解决循环依赖**

```java
--------------------------------------------------------------------
//暴露早期对象到缓存中用于解决依赖的。
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized(this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }

    }
}
```

#### populateBean()

**给创建的**bean**进行赋值** 

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
    //从bean定义中获取属性列表
    PropertyValues pvs = mbd.getPropertyValues();
    if (bw == null) {
        if (!((PropertyValues)pvs).isEmpty()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
    } else {
        boolean continueWithPropertyPopulation = true;
        if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
            Iterator var6 = this.getBeanPostProcessors().iterator();

            while(var6.hasNext()) {
                BeanPostProcessor bp = (BeanPostProcessor)var6.next();
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                    if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                        continueWithPropertyPopulation = false;
                        break;
                    }
                }
            }
        }

        /** 
         * 在属性被填充前，给 InstantiationAwareBeanPostProcessor 类型的后置处理器一个修改
		* bean 状态的机会。官方的解释是：让用户可以自定义属性注入。比如用户实现一 
		* 个 InstantiationAwareBeanPostProcessor 类型的后置处理器，并通过 
		* postProcessAfterInstantiation 方法向 bean 的成员变量注入自定义的信息。当然，如果无 
		* 特殊需求，直接使用配置中的信息注入即可。 
		*/
        if (continueWithPropertyPopulation) {
            if (mbd.getResolvedAutowireMode() == 1 || mbd.getResolvedAutowireMode() == 2) {
                MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
                if (mbd.getResolvedAutowireMode() == 1) {
                    this.autowireByName(beanName, mbd, bw, newPvs);
                }

                if (mbd.getResolvedAutowireMode() == 2) {
                    this.autowireByType(beanName, mbd, bw, newPvs);
                }

                pvs = newPvs;
            }

            boolean hasInstAwareBpps = this.hasInstantiationAwareBeanPostProcessors();
            boolean needsDepCheck = mbd.getDependencyCheck() != 0;
            /** 
            * 这里又是一种后置处理，用于在 Spring 填充属性到 bean 对象前，对属性的值进行相应的处理， 
            * 比如可以修改某些属性的值。这时注入到 bean 中的值就不是配置文件中的内容了， 
            * 而是经过后置处理器修改后的内容
			*/
            if (hasInstAwareBpps || needsDepCheck) {
                //过滤出所有需要进行依赖检查的属性编辑器 并且进行缓存起来
                PropertyDescriptor[] filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                //通过后置处理器来修改属性
                if (hasInstAwareBpps) {
                    Iterator var9 = this.getBeanPostProcessors().iterator();

                    while(var9.hasNext()) {
                        BeanPostProcessor bp = (BeanPostProcessor)var9.next();
                        if (bp instanceof InstantiationAwareBeanPostProcessor) {
                            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                            pvs = ibp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
                            if (pvs == null) {
                                return;
                            }
                        }
                    }
                }

                //需要检查的化 ，那么需要检查依赖
                if (needsDepCheck) {
                    this.checkDependencies(beanName, mbd, filteredPds, (PropertyValues)pvs);
                }
            }
			//设置属性到beanWapper中
            this.applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
        }
    }
}
//上诉代码的作用 
1)获取了bw的属性列表 
2)在属性列表中被填充的之前，通过InstantiationAwareBeanPostProcessor 对bw的属性进行修改 
3)判断自动装配模型来判断是调用byTypeh还是byName 
4）再次应用后置处理，用于动态修改属性列表 pvs 的内容 
5）把属性设置到bw中

------------------------------------------------------------------------------
protected void autowireByName(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
        /** 
        * spring认为的简单属性 
        * 1. CharSequence 接口的实现类，比如 String 
        * 2. Enum 
        * 3. Date 
        * 4. URI/URL 
        * 5. Number 的继承类，比如 Integer/Long 
        * 6. byte/short/int... 等基本类型 
        * 7. Locale 
        * 8. 以上所有类型的数组形式，比如 String[]、Date[]、int[] 等等 
        * 不包含在当前bean的配置文件的中属性 !pvs.contains(pd.getName() 
        * */
        String[] propertyNames = this.unsatisfiedNonSimpleProperties(mbd, bw);
        String[] var6 = propertyNames;
        int var7 = propertyNames.length;
		//循环解析出来的属性名称
        for(int var8 = 0; var8 < var7; ++var8) {
            String propertyName = var6[var8];
            //若当前循环的属性名称是当前bean中定义的属性
            if (this.containsBean(propertyName)) {
            	//去ioc中获取指定的bean对象
                Object bean = this.getBean(propertyName);
                //并且设置到当前bean中的pvs中
                pvs.add(propertyName, bean);
                //注册属性依赖
                this.registerDependentBean(propertyName, beanName);
                if (this.logger.isDebugEnabled()) {
                    this.logger.debug("Added autowiring by name from bean name '" + beanName + "' via property '" + propertyName + "' to bean named '" + propertyName + "'");
                }
            } else if (this.logger.isTraceEnabled()) {
                this.logger.trace("Not autowiring property '" + propertyName + "' of bean '" + beanName + "' by name: no matching bean found");
            }
        }

    }
--------------------------------------------------------------------------------------
protected void autowireByType(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {
		//获取自定义类型的转换器
        TypeConverter converter = this.getCustomTypeConverter();
        //没有获取到 把bw赋值给转换器(BeanWrapper实现了 TypeConverter接口 )
        if (converter == null) {
            converter = bw;
        }

        Set<String> autowiredBeanNames = new LinkedHashSet(4);
        //获取非简单bean的属性
        String[] propertyNames = this.unsatisfiedNonSimpleProperties(mbd, bw);
        String[] var8 = propertyNames;
        int var9 = propertyNames.length;
		//循环属性
        for(int var10 = 0; var10 < var9; ++var10) {
            String propertyName = var8[var10];

            try {
            //bw中是否有该属性的描述器
                PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);
                //若是Object的属性 不做解析
                if (Object.class != pd.getPropertyType()) {
               		 /* 
               		 * 获取 setter 方法（write method）的参数信息，比如参数在参数列表中的 
               		 * 位置，参数类型，以及该参数所归属的方法等信息 
               		 */
                    MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);
                    boolean eager = !PriorityOrdered.class.isAssignableFrom(bw.getWrappedClass());
                   // 创建依赖描述对象
                   DependencyDescriptor desc = new AbstractAutowireCapableBeanFactory.AutowireByTypeDependencyDescriptor(methodParam, eager);
                    Object autowiredArgument = this.resolveDependency(desc, beanName, autowiredBeanNames, (TypeConverter)converter);
                    if (autowiredArgument != null) {
                   	 	// 将解析出的 bean 存入到属性值列表（pvs）中
                        pvs.add(propertyName, autowiredArgument);
                    }

                    Iterator var17 = autowiredBeanNames.iterator();

                    while(var17.hasNext()) {
                        String autowiredBeanName = (String)var17.next();
                        this.registerDependentBean(autowiredBeanName, beanName);
                        if (this.logger.isDebugEnabled()) {
                            this.logger.debug("Autowiring by type from bean name '" + beanName + "' via property '" + propertyName + "' to bean named '" + autowiredBeanName + "'");
                        }
                    }

                    autowiredBeanNames.clear();
                }
            } catch (BeansException var19) {
                throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, var19);
            }
        }

    }
    
```

#### resolveReference()

真正的解析bean的依赖引用

```java

```



#### initializeBean()

**对** bean**进行初始化** 

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged(new PrivilegedAction<Object>() {
            public Object run() {
                AbstractAutowireCapableBeanFactory.this.invokeAwareMethods(beanName, bean);
                return null;
            }
        }, this.getAccessControlContext());
    } else {
        //调用bean实现的 XXXAware接口
        this.invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 调用bean的后置处理器的before 方法
        wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    }

    try {
        
        this.invokeInitMethods(beanName, wrappedBean, mbd);
    } catch (Throwable var6) {
        throw new BeanCreationException(mbd != null ? mbd.getResourceDescription() : null, beanName, "Invocation of init method failed", var6);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        //调用Bean的后置处理器的post方法
        wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
--------------------------------------------------------------------------
private void invokeAwareMethods(String beanName, Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware)bean).setBeanName(beanName);
        }

        if (bean instanceof BeanClassLoaderAware) {
            ((BeanClassLoaderAware)bean).setBeanClassLoader(this.getBeanClassLoader());
        }

        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware)bean).setBeanFactory(this);
        }
    }

}
```



## spring AOP源码解析

[参考1](https://www.cnblogs.com/toby-xu/p/11444288.html)

### @EnableAspectJAutoProxy

**凡是实现了ImportBeanDefinitionRegistrar可以给我们容器中添加bean定义信息**

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        
        //往容器中注册对应的 aspectj注解自动代理创建器
		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
        
		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}

}
============AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);===========
 public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry) {
    return registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry, null);
}

//注册一个AnnotationAwareAspectJAutoProxyCreator（注解适配的切面自动创建器）
public static BeanDefinition registerAspectJAnnotationAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
    return registerOrEscalateApcAsRequired(AnnotationAwareAspectJAutoProxyCreator.class, registry, source);
}

private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");

    //判断容器中有没有org.springframework.aop.config.internalAutoProxyCreator 名称的bean定义
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            if (currentPriority < requiredPriority) {
                apcDefinition.setBeanClassName(cls.getName());
            }
        }
        return null;
    }

    //容器中没有 那么就注册一个名称叫org.springframework.aop.config.internalAutoProxyCreator  类型是AnnotationAwareAspectJAutoProxyCreator的bean定义
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}	   
```

### **AnnotationAwareAspectJAutoProxyCreator**  

#### **BeanFactoryAware** 接口做了什么工作

**①:org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator 实现了BeanFactoryAware**

**我们查看源码的时候发现AbstractAutoProxyCreator 的setBeanFactory（）方法啥都没有做，但是又被子类覆盖了**

```java
@Override
public void setBeanFactory(BeanFactory beanFactory) {
    this.beanFactory = beanFactory;
}
```

**②:AbstractAdvisorAutoProxyCreator覆盖了AbstractAutoProxyCreator.setBeanFactory()方法做了二件事情**

1：调用父类的super.setBeanFactory(beanFactory);

2：调用本来的initBeanFactory((ConfigurableListableBeanFactory) beanFactory);初始化bean工厂方法

​     但是本类的AbstractAdvisorAutoProxyCreator.initBeanFactory()又被子类覆盖了

```java
public void setBeanFactory(BeanFactory beanFactory) {
    //调用父类AbstractAutoProxyCreator.setBeanFactory()方法
    super.setBeanFactory(beanFactory);
    if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
        throw new IllegalArgumentException(
            "AdvisorAutoProxyCreator requires a ConfigurableListableBeanFactory: " + beanFactory);
    }
    //初始化bean工程
    initBeanFactory((ConfigurableListableBeanFactory) beanFactory);
}

protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    this.advisorRetrievalHelper = new BeanFactoryAdvisorRetrievalHelperAdapter(beanFactory);
}	
```

**③:AnnotationAwareAspectJAutoProxyCreator#initBeanFactory覆盖了AbstractAdvisorAutoProxyCreator.initBeanFactory()方法**

```java
//创建一个aop的增强器通过@Apsectj注解的方式.
protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    //调用父类的
    super.initBeanFactory(beanFactory);
    //若 apsectj的增强器工厂对象为空,我们就创建一个ReflectiveAspectJAdvisorFactory
    if (this.aspectJAdvisorFactory == null) {
        this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
    }
    //不为空 我们就把aspectJAdvisorFactory 包装为BeanFactoryAspectJAdvisorsBuilderAdapter
    this.aspectJAdvisorsBuilder =
        new BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
}
```

**总结：AnnotationAwareAspectJAutoProxyCreator  实现了BeanFactoryAware 也是做了二个事情**

**事情1:把Beanfactory 保存到AnnotationAwareAspectJAutoProxyCreator  组件上.**

**事情2: 为AnnotationAwareAspectJAutoProxyCreator 的aspectJAdvisorsBuilder  aspect增强器构建器赋值**

***

#### **BeanPostProcessor接口（后置处理器的特性）**

①:postProcessBeforeInitialization初始化之前的方法 貌似什么都没有干

```java
public Object postProcessBeforeInitialization(Object bean, String beanName) {
    return bean;
}
```

**②:postProcessAfterInitialization 这个方法很重要 很重要 很重要 很重要很重要 很重要很重要 很重要很重要 很重要 后面单独说(创建代理对象的逻辑)**

```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            //包装bean 真正的创建代理对象逻辑
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

#### **InstantiationAwareBeanPostProcessor**

**(后置处理器的一种,在实例化之前进行调用)**

**追根溯源 AbstractAutoProxyCreator类实现了SmartInstantiationAwareBeanPostProcessor接口 所以我们分析SmartInstantiationAwareBeanPostProcessor的二个方法**

**①postProcessBeforeInstantiation方法**

```java
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
    Object cacheKey = getCacheKey(beanClass, beanName);

    // 判断TargetSource缓存中是否包含当前bean，如果不包含，则判断当前bean是否是已经被代理的bean，
    // 如果代理过，则不对当前传入的bean进行处理，如果没代理过，则判断当前bean是否为系统bean，或者是
    // 切面逻辑不会包含的bean，如果是，则将当前bean缓存到advisedBeans中，否则继续往下执行。
    // 经过这一步的处理之后，只有在TargetSource中没有进行缓存，并且应该被切面逻辑环绕，但是目前还未
    // 生成代理对象的bean才会通过此方法。

    if (beanName == null || !this.targetSourcedBeans.contains(beanName)) {

        if (this.advisedBeans.containsKey(cacheKey)) {
            return null;
        }
        //若是基础的class ||或者是否应该跳过  shouldSkip直接返回false
        if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
            //把cacheKey 存放在advisedBeans中
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            //返回null
            return null;
        }
    }

    // 获取封装当前bean的TargetSource对象，如果不存在，则直接退出当前方法，否则从TargetSource
    // 中获取当前bean对象，并且判断是否需要将切面逻辑应用在当前bean上。
    if (beanName != null) {
        TargetSource targetSource = getCustomTargetSource(beanClass, beanName);
        if (targetSource != null) {
            this.targetSourcedBeans.add(beanName);
            //// 获取能够应用当前bean的切面逻辑
            Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(beanClass, beanName, targetSource);
            //// 根据切面逻辑为当前bean生成代理对象
            Object proxy = createProxy(beanClass, beanName, specificInterceptors, targetSource);
            this.proxyTypes.put(cacheKey, proxy.getClass());
            return proxy;
        }
    }

    return null;
}

=============================判断是不是基础的bean======================================= 
    protected boolean isInfrastructureClass(Class<?> beanClass) {
    //是不是Advice PointCut  Advisor   AopInfrastructureBean  满足任意返回ture
    boolean retVal = Advice.class.isAssignableFrom(beanClass) ||
        Pointcut.class.isAssignableFrom(beanClass) ||
            Advisor.class.isAssignableFrom(beanClass) ||
                AopInfrastructureBean.class.isAssignableFrom(beanClass);
    if (retVal && logger.isTraceEnabled()) {
        logger.trace("Did not attempt to auto-proxy infrastructure class [" + beanClass.getName() + "]");
    }
    return retVal;
}	


```

### BeanPostProcessor

**真正的创建代理对象从BeanPostProcessor处理器的后置方法开始**

```
1:>org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization

2:>org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary  有必要的话进行包装

3:>org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#getAdvicesAndAdvisorsForBean

4:>org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#findEligibleAdvisors  

5:>org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#findAdvisorsThatCanApply  

6:>org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy创建代理对象
```

#### **postProcessAfterInitialization**

```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    if (bean != null) {
        //通过传入的class 和beanName生成缓存key
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            //若当前bean合适被包装为代理bean就进行处理
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

#### **wrapIfNecessary**

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    //已经被处理过的 不进行下面的处理
    if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    //不需要被增强的直接返回
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    //判断当前bean是不是基础类型的bean,或者指定类型的bean 不需要代理
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }

    //获取通知或者增强器
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    //获取的不为空，生成代理对象
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        //创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }
    //加入advisedBeans集合中 
    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}

/**
 * 判断什么是基础的class
 * */
protected boolean isInfrastructureClass(Class<?> beanClass) {
		//判断当前的class是不是 Pointcut Advisor   Advice  AopInfrastructureBean 只要有一个满足就返回true
    boolean retVal = Advice.class.isAssignableFrom(beanClass) ||
        Pointcut.class.isAssignableFrom(beanClass) ||
            Advisor.class.isAssignableFrom(beanClass) ||
                AopInfrastructureBean.class.isAssignableFrom(beanClass);
    if (retVal && logger.isTraceEnabled()) {
        logger.trace("Did not attempt to auto-proxy infrastructure class [" + beanClass.getName() + "]");
    }
    return retVal;
}
```

#### **getAdvicesAndAdvisorsForBean** 

```java
//找到符合条件的增强器 
@Override
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, TargetSource targetSource) {
    //查找符合条件的增强器
    List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
    if (advisors.isEmpty()) {
        return DO_NOT_PROXY;
    }
    return advisors.toArray();
}
```

#### **findEligibleAdvisors**  

```java
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
    //找到候选的增强器
    List<Advisor> candidateAdvisors = findCandidateAdvisors();
    //从候选的中选出能用的增强器
    List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
    extendAdvisors(eligibleAdvisors);
    if (!eligibleAdvisors.isEmpty()) {
        eligibleAdvisors = sortAdvisors(eligibleAdvisors);
    }
    return eligibleAdvisors;
}
```

#### **findCandidateAdvisors 从IOC容器中查找所有的增强器**

```java
protected List<Advisor> findCandidateAdvisors() {
    //调用父类获取增强器
    List<Advisor> advisors = super.findCandidateAdvisors();
    //解析 @Aspect 注解，并构建通知器
    advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    return advisors;
}
	
	
======================================super.findCandidateAdvisors();==============================
    public List<Advisor> findAdvisorBeans() {
    //先从缓存中获取增强器   cachedAdvisorBeanNames是advisor的名称
    String[] advisorNames = this.cachedAdvisorBeanNames;
    //缓存中没有获取到
    if (advisorNames == null) {
        //从IOC容器中获取增强器的名称
        advisorNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
            this.beanFactory, Advisor.class, true, false);
        //赋值给增强器缓存
        this.cachedAdvisorBeanNames = advisorNames;
    }
    //在IOC容器中没有获取到直接返回
    if (advisorNames.length == 0) {
        return new ArrayList<Advisor>();
    }

    List<Advisor> advisors = new ArrayList<Advisor>();
    //遍历所有的增强器
    for (String name : advisorNames) {
        if (isEligibleBean(name)) {
            //忽略正在创建的增强器
            if (this.beanFactory.isCurrentlyInCreation(name)) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Skipping currently created advisor '" + name + "'");
                }
            }
            else {
                try {
                    //通过getBean的形式创建增强器 //并且将bean 添加到advisors中
                    advisors.add(this.beanFactory.getBean(name, Advisor.class));
                }
                catch (BeanCreationException ex) {
                    Throwable rootCause = ex.getMostSpecificCause();
                    if (rootCause instanceof BeanCurrentlyInCreationException) {
                        BeanCreationException bce = (BeanCreationException) rootCause;
                        if (this.beanFactory.isCurrentlyInCreation(bce.getBeanName())) {
                            if (logger.isDebugEnabled()) {
                                logger.debug("Skipping advisor '" + name +
                                             "' with dependency on currently created bean: " + ex.getMessage());
                            }
                            // Ignore: indicates a reference back to the bean we're trying to advise.
                            // We want to find advisors other than the currently created bean itself.
                            continue;
                        }
                    }
                    throw ex;
                }
            }
        }
    }
    return advisors;
}
	
=============================================aspectJAdvisorsBuilder.buildAspectJAdvisors()解析@Aspject的=======================================	
下面buildAspectJAdvisors这个方法为我们做了什么？ 
第一步:先从增强器缓存中获取增强器对象
  判断缓存中有没有增强器对象,有，那么直接从缓存中直接获取返回出去
  没有.....从容器中获取所有的beanName
  遍历上一步获取所有的beanName,通过beanName获取beanType
  根据beanType判断当前bean是否是一个的Aspect注解类，若不是则不做任何处理
  调用advisorFactory.getAdvisors获取通知器

  

	public List<Advisor> buildAspectJAdvisors() {
		//先从缓存中获取
		List<String> aspectNames = this.aspectBeanNames;
		//缓存中没有获取到 //缓存字段aspectNames没有值 注意实例化第一个单实例bean的时候就会触发解析切面
		if (aspectNames == null) {
			synchronized (this) {
			    //在尝试从缓存中获取一次
				aspectNames = this.aspectBeanNames;
				//还是没有获取到
				if (aspectNames == null) {
					//从容器中获取所有的bean的name 
					List<Advisor> advisors = new LinkedList<Advisor>();
					 //用于保存切面的名称的集合
					aspectNames = new LinkedList<String>();
					/**
                     * AOP功能中在这里传入的是Object对象，代表去容器中获取到所有的组件的名称，然后再
                     * 进行遍历，这个过程是十分的消耗性能的，所以说Spring会再这里加入了保存切面信息的缓存。
                     * 但是事务功能不一样，事务模块的功能是直接去容器中获取Advisor类型的，选择范围小，且不消耗性能。
                     * 所以Spring在事务模块中没有加入缓存来保存我们的事务相关的advisor
                     */
					String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
							this.beanFactory, Object.class, true, false);
					
					 //遍历我们从IOC容器中获取处的所有Bean的名称	
					for (String beanName : beanNames) {
						if (!isEligibleBean(beanName)) {
							continue;
						}
						//根据beanName获取bean的类型
						Class<?> beanType = this.beanFactory.getType(beanName);
						if (beanType == null) {
							continue;
						}
						 //根据class对象判断是不是切面 @Aspect
						if (this.advisorFactory.isAspect(beanType)) {
                           		//是切面类
                            	//加入到缓存中
							aspectNames.add(beanName);
							//把beanName和class对象构建成为一个AspectMetadata
							AspectMetadata amd = new AspectMetadata(beanType, beanName);
							if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
								MetadataAwareAspectInstanceFactory factory =
										new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
								//从aspectj中获取通知器
								List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
								if (this.beanFactory.isSingleton(beanName)) {
									this.advisorsCache.put(beanName, classAdvisors);
								}
								else {
									this.aspectFactoryCache.put(beanName, factory);
								}
								advisors.addAll(classAdvisors);
							}
							else {
								// Per target or per this.
								if (this.beanFactory.isSingleton(beanName)) {
									throw new IllegalArgumentException("Bean with name '" + beanName +
											"' is a singleton, but aspect instantiation model is not singleton");
								}
								MetadataAwareAspectInstanceFactory factory =
										new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
								this.aspectFactoryCache.put(beanName, factory);
								advisors.addAll(this.advisorFactory.getAdvisors(factory));
							}
						}
					}
					this.aspectBeanNames = aspectNames;
					return advisors;
				}
			}
		}
        
        //返回空
		if (aspectNames.isEmpty()) {
			return Collections.emptyList();
		}
		//缓存中有增强器，我们从缓存中获取返回出去
		List<Advisor> advisors = new LinkedList<Advisor>();
		for (String aspectName : aspectNames) {
			List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
			if (cachedAdvisors != null) {
				advisors.addAll(cachedAdvisors);
			}
			else {
				MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
				advisors.addAll(this.advisorFactory.getAdvisors(factory));
			}
		}
		return advisors;
	}
	

//获取通知	
===========org.springframework.aop.aspectj.annotation.AspectJAdvisorFactory#getAdvisors========
/**
 * 
 * 
 * */

	public List<Advisor> getAdvisors(MetadataAwareAspectInstanceFactory aspectInstanceFactory) {
		//获取标识了@AspectJ标志的切面类
		Class<?> aspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
		//获取切面的名称
		String aspectName = aspectInstanceFactory.getAspectMetadata().getAspectName();
		validate(aspectClass);

		// We need to wrap the MetadataAwareAspectInstanceFactory with a decorator
		// so that it will only instantiate once.
		MetadataAwareAspectInstanceFactory lazySingletonAspectInstanceFactory =
				new LazySingletonAspectInstanceFactoryDecorator(aspectInstanceFactory);
        
		List<Advisor> advisors = new ArrayList<Advisor>();
		//获取切面类排除@PointCut标志的所有方法
		for (Method method : getAdvisorMethods(aspectClass)) {
			//每一个方法都调用getAdvisor方法来获取增强器
			Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, advisors.size(), aspectName);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		// If it's a per target aspect, emit the dummy instantiating aspect.
		if (!advisors.isEmpty() && lazySingletonAspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			Advisor instantiationAdvisor = new SyntheticInstantiationAdvisor(lazySingletonAspectInstanceFactory);
			advisors.add(0, instantiationAdvisor);
		}

		// Find introduction fields.
		for (Field field : aspectClass.getDeclaredFields()) {
			Advisor advisor = getDeclareParentsAdvisor(field);
			if (advisor != null) {
				advisors.add(advisor);
			}
		}

		return advisors;
	}
	
	
//通过方法获取增强器	
public Advisor getAdvisor(Method candidateAdviceMethod, MetadataAwareAspectInstanceFactory aspectInstanceFactory,
			int declarationOrderInAspect, String aspectName) {

		validate(aspectInstanceFactory.getAspectMetadata().getAspectClass());
        
        //获取aspectj的切点表达式
		AspectJExpressionPointcut expressionPointcut = getPointcut(
				candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
		if (expressionPointcut == null) {
			return null;
		}
        
        //创建advisor实现类
		return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
				this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
}

//获取切点表达式
private AspectJExpressionPointcut getPointcut(Method candidateAdviceMethod, Class<?> candidateAspectClass) {
		//获取切面注解 @Before   @After。。。。。。
		AspectJAnnotation<?> aspectJAnnotation =
				AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
		if (aspectJAnnotation == null) {
			return null;
		}
        
        //获取切点表达式对象
		AspectJExpressionPointcut ajexp =
				new AspectJExpressionPointcut(candidateAspectClass, new String[0], new Class<?>[0]);
		//设置切点表达式
		ajexp.setExpression(aspectJAnnotation.getPointcutExpression());
		ajexp.setBeanFactory(this.beanFactory);
		return ajexp;
	}
	
//找到切面类中方法上的切面注解	
protected static AspectJAnnotation<?> findAspectJAnnotationOnMethod(Method method) {
        //Pointcut.class, Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class
		for (Class<?> clazz : ASPECTJ_ANNOTATION_CLASSES) {
			AspectJAnnotation<?> foundAnnotation = findAnnotation(method, (Class<Annotation>) clazz);
			if (foundAnnotation != null) {
				return foundAnnotation;
			}
		}
		return null;
	}
	
//把切点，候选的方法....统一处理生成一个增强器
public InstantiationModelAwarePointcutAdvisorImpl(AspectJExpressionPointcut declaredPointcut,
			Method aspectJAdviceMethod, AspectJAdvisorFactory aspectJAdvisorFactory,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {

		//当前的切点表达式
        this.declaredPointcut = declaredPointcut;
        //切面的class对象
        this.declaringClass = aspectJAdviceMethod.getDeclaringClass();
        //切面方法的名称
        this.methodName = aspectJAdviceMethod.getName();
        //切面方法的参数类型
        this.parameterTypes = aspectJAdviceMethod.getParameterTypes();
        //切面方法对象
        this.aspectJAdviceMethod = aspectJAdviceMethod;
        //aspectj的通知工厂
        this.aspectJAdvisorFactory = aspectJAdvisorFactory;
        //aspect的实例工厂
        this.aspectInstanceFactory = aspectInstanceFactory;
        //切面的顺序
        this.declarationOrder = declarationOrder;
        //切面的名称
        this.aspectName = aspectName;

    	 /**
         * 判断当前的切面对象是否需要延时加载
         */
		if (aspectInstanceFactory.getAspectMetadata().isLazilyInstantiated()) {
			// Static part of the pointcut is a lazy type.
			Pointcut preInstantiationPointcut = Pointcuts.union(
					aspectInstanceFactory.getAspectMetadata().getPerClausePointcut(), this.declaredPointcut);

			// Make it dynamic: must mutate from pre-instantiation to post-instantiation state.
			// If it's not a dynamic pointcut, it may be optimized out
			// by the Spring AOP infrastructure after the first evaluation.
			this.pointcut = new PerTargetInstantiationModelPointcut(
					this.declaredPointcut, preInstantiationPointcut, aspectInstanceFactory);
			this.lazy = true;
		}
		else {
			// A singleton aspect.
			this.pointcut = this.declaredPointcut;
			this.lazy = false;
			//实例化切面 //将切面中的通知构造为advice通知对象
			this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
		}
	}
	
//获取advice 切面对象	
public Advice getAdvice(Method candidateAdviceMethod, AspectJExpressionPointcut expressionPointcut,
			MetadataAwareAspectInstanceFactory aspectInstanceFactory, int declarationOrder, String aspectName) {
        
    //获取候选的切面类
    Class<?> candidateAspectClass = aspectInstanceFactory.getAspectMetadata().getAspectClass();
    validate(candidateAspectClass);

    //获取切面注解
    AspectJAnnotation<?> aspectJAnnotation =
        AbstractAspectJAdvisorFactory.findAspectJAnnotationOnMethod(candidateAdviceMethod);
    //解析出来的注解信息是否为null	
    if (aspectJAnnotation == null) {
        return null;
    }

    // If we get here, we know we have an AspectJ method.
    // Check that it's an AspectJ-annotated class
    if (!isAspect(candidateAspectClass)) {
        throw new AopConfigException("Advice must be declared inside an aspect type: " +
                                     "Offending method '" + candidateAdviceMethod + "' in class [" +
                                     candidateAspectClass.getName() + "]");
    }

    if (logger.isDebugEnabled()) {
        logger.debug("Found AspectJ method: " + candidateAdviceMethod);
    }

    AbstractAspectJAdvice springAdvice;

    //判断注解的类型
    switch (aspectJAnnotation.getAnnotationType()) {
            //是切点的返回null
        case AtPointcut:
            if (logger.isDebugEnabled()) {
                logger.debug("Processing pointcut '" + candidateAdviceMethod.getName() + "'");
            }
            return null;
            //是不是环绕通知
        case AtAround:
            springAdvice = new AspectJAroundAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
            //是不是前置通知	
        case AtBefore:
            springAdvice = new AspectJMethodBeforeAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
            //是不是后置通知
        case AtAfter:
            springAdvice = new AspectJAfterAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            break;
            //返回通知
        case AtAfterReturning:
            springAdvice = new AspectJAfterReturningAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterReturning afterReturningAnnotation = (AfterReturning) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterReturningAnnotation.returning())) {
                springAdvice.setReturningName(afterReturningAnnotation.returning());
            }
            break;
            // 是不是异常通知	
                case AtAfterThrowing:
            springAdvice = new AspectJAfterThrowingAdvice(
                candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
            AfterThrowing afterThrowingAnnotation = (AfterThrowing) aspectJAnnotation.getAnnotation();
            if (StringUtils.hasText(afterThrowingAnnotation.throwing())) {
                springAdvice.setThrowingName(afterThrowingAnnotation.throwing());
            }
            break;
        default:
            throw new UnsupportedOperationException(
                "Unsupported advice type on method: " + candidateAdviceMethod);
    }

    // Now to configure the advice...
    springAdvice.setAspectName(aspectName);
    springAdvice.setDeclarationOrder(declarationOrder);
    /*
         * 获取方法的参数列表名称，比如方法 int sum(int numX, int numY), 
         * getParameterNames(sum) 得到 argNames = [numX, numY]
         */
    String[] argNames = this.parameterNameDiscoverer.getParameterNames(candidateAdviceMethod);
    if (argNames != null) {
        //为切面设置参数
        springAdvice.setArgumentNamesFromStringArray(argNames);
    }
    springAdvice.calculateArgumentBindings();

    return springAdvice;
}	
```

#### **findAdvisorsThatCanApply** 

```java
//获取能够使用的增强器
protected List<Advisor> findAdvisorsThatCanApply(
    List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

    ProxyCreationContext.setCurrentProxiedBeanName(beanName);
    try {
        return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
    }
    finally {
        ProxyCreationContext.setCurrentProxiedBeanName(null);
    }
}

//获取能使用的增强器
public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
    if (candidateAdvisors.isEmpty()) {
        return candidateAdvisors;
    }
    List<Advisor> eligibleAdvisors = new LinkedList<Advisor>();
    //遍历候选的增强器 把他增加到eligibleAdvisors集合中返回
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
            eligibleAdvisors.add(candidate);
        }
    }
    boolean hasIntroductions = !eligibleAdvisors.isEmpty();
    for (Advisor candidate : candidateAdvisors) {
        if (candidate instanceof IntroductionAdvisor) {
            // already processed
            continue;
        }
        if (canApply(candidate, clazz, hasIntroductions)) {
            eligibleAdvisors.add(candidate);
        }
    }
    return eligibleAdvisors;
}	

//判断是当前的增强器是否能用 通过方法匹配来计算当前是否合适当前类的增强器
public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
    if (advisor instanceof IntroductionAdvisor) {
        return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
    }
    else if (advisor instanceof PointcutAdvisor) {
        PointcutAdvisor pca = (PointcutAdvisor) advisor;
        return canApply(pca.getPointcut(), targetClass, hasIntroductions);
    }
    else {
        // It doesn't have a pointcut so we assume it applies.
        return true;
    }
}



public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
    Assert.notNull(pc, "Pointcut must not be null");
    if (!pc.getClassFilter().matches(targetClass)) {
        return false;
    }

    //创建一个方法匹配器
    MethodMatcher methodMatcher = pc.getMethodMatcher();
    if (methodMatcher == MethodMatcher.TRUE) {
        // No need to iterate the methods if we're matching any method anyway...
        return true;
    }

    //包装方法匹配器
    IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
    if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
        introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
    }

    //获取本来和接口
    Set<Class<?>> classes = new LinkedHashSet<Class<?>>(ClassUtils.getAllInterfacesForClassAsSet(targetClass));
    classes.add(targetClass);
    //循环classes
    for (Class<?> clazz : classes) {
        //获取所有的方法 进行匹配
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
        for (Method method : methods) {
            if ((introductionAwareMethodMatcher != null &&
                 introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions)) ||
                methodMatcher.matches(method, targetClass)) {
                return true;
            }
        }
    }

    return false;
}	
```

#### **createProxy**

**创建代理对象**

```java
protected Object createProxy(
			Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
        
    //判断容器的类型ConfigurableListableBeanFactory
    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }

    //创建代理工程
    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);


    /*
         * 默认配置下，或用户显式配置 proxy-target-class = "false" 时，
         * 这里的 proxyFactory.isProxyTargetClass() 也为 false
         */
    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }

        else {
            /*
             * 检测 beanClass 是否实现了接口，若未实现，则将 
             * proxyFactory 的成员变量 proxyTargetClass 设为 true
             */
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    //获取容器中的方法增强器
    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }

    //创建代理对象
    return proxyFactory.getProxy(getProxyClassLoader());
}

public Object getProxy(ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}

public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " +
                                         "Either an interface or a target is required for proxy creation.");
        }
        //是否实现了接口
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            //jdk代理
            return new JdkDynamicAopProxy(config);
        }
        //cglib代理
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        jdk代理
            return new JdkDynamicAopProxy(config);
    }
}

public Object getProxy(ClassLoader classLoader) {
    if (logger.isDebugEnabled()) {
        logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
    }
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    //创建jdk代理对象
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}

```



### invoke

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		MethodInvocation invocation;
		Object oldProxy = null;
		boolean setProxyContext = false;

		TargetSource targetSource = this.advised.targetSource;
		Class<?> targetClass = null;
		Object target = null;

		try {
		  
			Object retVal;
            
            //是否暴露代理对象
			if (this.advised.exposeProxy) {
				//把代理对象添加到TheadLocal中
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}

            //获取被代理对象
			target = targetSource.getTarget();
			if (target != null) {
			    //设置被代理对象的class
				targetClass = target.getClass();
			}

			//把增强器转为方法拦截器链
			List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

		    //若方法拦截器链为空
			if (chain.isEmpty()) {
                //通过反射直接调用目标方法
				Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
				retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
			}
			else {
				//创建方法拦截器调用链条
				invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
				//执行拦截器链
				retVal = invocation.proceed();
			}

			//获取方法的返回值类型
			Class<?> returnType = method.getReturnType();
			if (retVal != null && retVal == target &&
					returnType != Object.class && returnType.isInstance(proxy) &&
					!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
				//如果方法返回值为 this，即 return this; 则将代理对象 proxy 赋值给 retVal 
				retVal = proxy;
			}
			//如果返回值类型为基础类型，比如 int，long 等，当返回值为 null，抛出异常
			else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
				throw new AopInvocationException(
						"Null return value from advice does not match primitive return type for: " + method);
			}
			return retVal;
		}
		finally {
			if (target != null && !targetSource.isStatic()) {
				// Must have come from TargetSource.
				targetSource.releaseTarget(target);
			}
			if (setProxyContext) {
				// Restore old proxy.
				AopContext.setCurrentProxy(oldProxy);
			}
		}
	}
------------org.springframework.aop.framework.ReflectiveMethodInvocation#proceed----------------
    //该方法的调用用到了递归和责任链设计模式：
public Object proceed() throws Throwable {
     //从-1开始,下标=拦截器的长度-1的条件满足表示执行到了最后一个拦截器的时候，此时执行目标方法
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        return this.invokeJoinpoint();
    } else {
        //获取第一个方法拦截器使用的是前++
        Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
        if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
            InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
            return dm.methodMatcher.matches(this.method, this.targetClass, this.arguments) ? dm.interceptor.invoke(this) : this.proceed();
        } else {
            return ((MethodInterceptor)interceptorOrInterceptionAdvice).invoke(this);
        }
    }
}
=====org.springframework.aop.framework.AdvisedSupport#getInterceptorsAndDynamicInterceptionAdvice====
把增强器中转为方法拦截器链
	public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, Class<?> targetClass) {
		//从缓存中获取缓存key 第一次肯定获取不到
		MethodCacheKey cacheKey = new MethodCacheKey(method);
		//通过cacheKey获取缓存值
		List<Object> cached = this.methodCache.get(cacheKey);
		
		//从缓存中没有获取到
		if (cached == null) {
		    //获取所有的拦截器
			cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
					this, method, targetClass);
		    //放入缓存.....
			this.methodCache.put(cacheKey, cached);
		}
		return cached;
	}

=====================org.springframework.aop.framework.AdvisorChainFactory#getInterceptorsAndDynamicInterceptionAdvice====
	public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
			Advised config, Method method, Class<?> targetClass) {

	    //创建拦截器集合长度是增强器的长度
		List<Object> interceptorList = new ArrayList<Object>(config.getAdvisors().length);
		
		Class<?> actualClass = (targetClass != null ? targetClass : method.getDeclaringClass());
		boolean hasIntroductions = hasMatchingIntroductions(config, actualClass);
		AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
        
        //遍历所有的增强器集合
		for (Advisor advisor : config.getAdvisors()) {
			//判断增强器是不是PointcutAdvisor
			if (advisor instanceof PointcutAdvisor) {
				//把增强器转为PointcutAdvisor
				PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
				//通过方法匹配器对增强器进行匹配
				if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(actualClass)) {
					MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
					//能够匹配
					if (MethodMatchers.matches(mm, method, actualClass, hasIntroductions)) {
						//把增强器转为拦截器
						MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
						if (mm.isRuntime()) {
							// Creating a new object instance in the getInterceptors() method
							// isn't a problem as we normally cache created chains.
							for (MethodInterceptor interceptor : interceptors) {
								interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
							}
						}
						else {
							interceptorList.addAll(Arrays.asList(interceptors));
						}
					}
				}
			}
			else if (advisor instanceof IntroductionAdvisor) {
				IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
				if (config.isPreFiltered() || ia.getClassFilter().matches(actualClass)) {
					Interceptor[] interceptors = registry.getInterceptors(advisor);
					interceptorList.addAll(Arrays.asList(interceptors));
				}
			}
			else {
				Interceptor[] interceptors = registry.getInterceptors(advisor);
				interceptorList.addAll(Arrays.asList(interceptors));
			}
		}

		return interceptorList;
	}
	
	
```

总结：通过@EnableAspectJAutoProxy注解开启AOP功能，该注解为我们Spring容器中注册了AnnotationAwareAspectJAuto ProxyCreator组件，AOP的准备和代理创建都在这个组件中完成，AnnotationAwareAspectJAutoProxyCreator继承AbstractAuto ProxyCreator实现了InstantiationAwareBeanPostProcessor接口，在方法postProcessBeforeInstantiation中找到Spring容器中所有的增强器，为创建代理做准备；AnnotationAwareAspectJAutoProxyCreator继承了AbstractAutoProxyCreator实现了BeanPost Processor接口，在方法postProcessAfterInitialization中通过前面找到的候选增强器中找到合适的增强器来创建代理对象，最后调用目标方法，进去到代理对象的invoke方法中进行调用。

## spring 事务管理

[参考1](https://www.cnblogs.com/toby-xu/p/11645162.html)

### **PlatformTransactionManager： （平台）事务管理器**

**Spring并不直接管理事务，而是提供了多种事务管理器** ，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。

**org.springframework.transaction.PlatformTransactionManager** ，

通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。

```java
public interface PlatformTransactionManager {
 /**
 *获取事物状态
 */
 TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
 /**
 *事物提交
 */
 void commit(TransactionStatus status) throws TransactionException;
 /**
 *事物回滚
 */
 void rollback(TransactionStatus status) throws TransactionException;
}
```





### **TransactionDefinition： 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则**

**org.springframework.transaction.TransactionDefinition**

TransactionDefinition接口中定义了5个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等的常量。
我下面只是列出了TransactionDefinition接口中的方法而没有给出接口中定义的常量，该接口中的常量信息会在后面依次介绍到

```java
public interface TransactionDefinition {

	/**
	 * 支持当前事物，若当前没有事物就创建一个事物
	 * */
	int PROPAGATION_REQUIRED = 0;

    /**
     *  如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行
     * */
	int PROPAGATION_SUPPORTS = 1;

	/**
     *如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常
	 */
	int PROPAGATION_MANDATORY = 2;

    /**
     *创建一个新的事务，如果当前存在事务，则把当前事务挂起
     **/
	int PROPAGATION_REQUIRES_NEW = 3;

    /**
     *  以非事务方式运行，如果当前存在事务，则把当前事务挂起
     * */
	int PROPAGATION_NOT_SUPPORTED = 4;

    /**
     * 以非事务方式运行，如果当前存在事务，则抛出异常。
     * */
	int PROPAGATION_NEVER = 5;

    /**
     *  表示如果当前正有一个事务在运行中，则该方法应该运行在 一个嵌套的事务中，
        被嵌套的事务可以独立于封装事务进行提交或者回滚(保存点)，
        如果封装事务不存在,行为就像 PROPAGATION_REQUIRES NEW
     * */
	int PROPAGATION_NESTED = 6;


	/**
     *使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别
	 */
	int ISOLATION_DEFAULT = -1;

	/**
     *最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
	 */
	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;

	/**
     *允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
	 */
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;

	/**
     *对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生
	 */
	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;

	/**
	 *最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，
	 * 也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能通常情况下也不会用到该级别
	 */
	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;


	/**
     *使用默认的超时时间
	 */
	int TIMEOUT_DEFAULT = -1;


	/**
     *获取事物的传播行为
	 */
	int getPropagationBehavior();

	/**
     *获取事物的隔离级别
	 */
	int getIsolationLevel();

	/**
     *返回事物的超时时间
	 */
	int getTimeout();

	/**
     *返回当前是否为只读事物
	 */
	boolean isReadOnly();

	/**
     *获取事物的名称
	 */
	@Nullable
	String getName();

}
```



### **TransactionStatus： 事务运行状态**

TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事物

```java
public interface TransactionStatus extends SavepointManager, Flushable {

    /**
     * 是否为新事物
     * */
	boolean isNewTransaction();

	/**
     *是否有保存点
	 */
	boolean hasSavepoint();

	/**
     *设置为只回滚
	 */
	void setRollbackOnly();

	/**
     *是否为只回滚
	 */
	boolean isRollbackOnly();

	/**
	 *属性
	 */
	@Override
	void flush();

	/**
     *判断当前事物是否已经完成
	 */
	boolean isCompleted();

}
​
```

### **@EnableTransactionManagement**

```java
org.springframework.transaction.annotation.EnableTransactionManagement

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
    
    /**
     * 指定使用什么代理模式(true为cglib代理,false 为jdk代理)
     * */
	boolean proxyTargetClass() default false;

    /**
     * 通知模式 是使用代理模式还是aspectj  我们一般使用Proxy
     * */
	AdviceMode mode() default AdviceMode.PROXY;

	int order() default Ordered.LOWEST_PRECEDENCE;
}​
----------------------------------------------------------------------
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

    /**
     * 往容器中添加组件 1) AutoProxyRegistrar
     *                  2) ProxyTransactionManagementConfiguration
     * */
	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {  //因为我们配置的默认模式是PROXY
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(),
						ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {
						TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}

}​
```

#### **AutoProxyRegistrar**

```java
public class AutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	private final Log logger = LogFactory.getLog(getClass());

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean candidateFound = false;
		//从我们传入进去的配置类上获取所有的注解的
		Set<String> annoTypes = importingClassMetadata.getAnnotationTypes();
		//循环我们上一步获取的注解
		for (String annoType : annoTypes) {
		    //获取注解的元信息
			AnnotationAttributes candidate = AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
			if (candidate == null) {
				continue;
			}
			//获取注解的mode属性
			Object mode = candidate.get("mode");
			//获取注解的proxyTargetClass
			Object proxyTargetClass = candidate.get("proxyTargetClass");
			//根据mode和proxyTargetClass的判断来注册不同的组件
			if (mode != null && proxyTargetClass != null && AdviceMode.class == mode.getClass() &&
					Boolean.class == proxyTargetClass.getClass()) {
				candidateFound = true;
				if (mode == AdviceMode.PROXY) {
					AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
					if ((Boolean) proxyTargetClass) {
						AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
						return;
					}
				}
			}
		}
	}

}

public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry) {
		return registerAutoProxyCreatorIfNecessary(registry, null);
	}
	
public static BeanDefinition registerAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
		return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);
	}
	
private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");

    //判断容器中有没有org.springframework.aop.config.internalAutoProxyCreator名字的bean定义,
    if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
        BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
        if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
            int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
            int requiredPriority = findPriorityForClass(cls);
            if (currentPriority < requiredPriority) {
                apcDefinition.setBeanClassName(cls.getName());
            }
        }
        return null;
    }


    //自己注册一个org.springframework.aop.config.internalAutoProxyCreator的组件
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
    beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
    return beanDefinition;
}

```

**AutoProxyRegistrar会为我们容器中导入了一个叫InfrastructureAdvisorAutoProxyCreator的组件**

1. #### 实现了InstantiationAwareBeanPostProcessor接口

   该接口有2个方法postProcessBeforeInstantiation和postProcessAfterInstantiation，其中实例化之前会执行postProcess BeforeInstantiation方法：

2. #### 实现了BeanPostProcessor接口

   　该接口有2个方法postProcessBeforeInitialization和postProcessAfterInitialization，其中组件初始化之后会执行postProcessAfterInitialization（该方法创建Aop和事务的代理对象）方法：

   


​     
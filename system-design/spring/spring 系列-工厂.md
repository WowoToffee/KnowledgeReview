# spring 系列-工厂

## 第一章 引言

1. EJB存在的问题

   1. 运行环境苛刻
   2. 代码移植性差

2. 什么是spring

   `spring是一个轻量级JavaEE解决方案，整合众多优秀的设计模式`

   - 轻量级

     ```
     运行环境没有要求
     代码移植性高
     ```

     

   - JavaEE的解决方案

     ```
     spring 能解决javaEE 上的所有问题,它是一个完整的体系：
     contorller：可以使用spring MVC解决
     service： 可以使用spring aop 代理解决
     DAO： 可以直接整个第三方框架 JPA/mybatis
     ```

   - 整合设计模式

     ```
     1. 工厂模式
     2. 代理模式
     3. 模板模式
     4. 策略模式
     ...
     ```

3. 设计模式

   ```
   1. 广义概念
   面向对象设计中，解决特定问题的经典代码
   2. 狭义概念
   人为定义的23种设计模式
   ```

4. 工厂设计模式

   1. 什么是工厂设计模式

      ```
      1.概念：通过工厂类，创建对象
      2.好处：解耦合
        耦合：指定是代码间的强关联关系，一方的改变会影响另一方
        问题：不利于代码的维护。
        	User user = new User();
      ```

   2. 简单工厂的设计

      ```java
      package com.yuziyan.basic;
      
      import java.io.IOException;
      import java.io.InputStream;
      import java.util.Properties;
      
      public class BeanFactory {
      
          private static Properties env = new Properties();
          static{
              try {
                  //第一步 获得IO输入流
                  InputStream inputStream = BeanFactory.class.getResourceAsStream("applicationContext.properties");
                  //第二步 文件内容 封装 Properties集合中 key = userService value = com.yuziyan.UserServiceImpl
                  env.load(inputStream);
              } catch (IOException e) {
                  e.printStackTrace();
              }
      
          }
      
          /*
      	   对象的创建方式：
      	       1. 直接调用构造方法 创建对象  UserService userService = new UserServiceImpl();
      	       2. 通过反射的形式 创建对象 解耦合
      	       Class clazz = Class.forName("com.yuziyan.basic.UserServiceImpl");
      	       UserService userService = (UserService)clazz.newInstance();
           */
      
          public static UserService getUserService(){
              UserService userService = null;
              try {
                  //                          com.yuziyan.basic.UserServiceImpl
                  Class clazz = Class.forName(env.getProperty("userService"));
                  userService = (UserService) clazz.newInstance();
              } catch (ClassNotFoundException e) {
                  e.printStackTrace();
              } catch (InstantiationException e) {
                  e.printStackTrace();
              } catch (IllegalAccessException e) {
                  e.printStackTrace();
              }
              return userService;
          }
      
          public static UserDao getUserDao(){
              UserDao userDao = null;
              try {
                  Class clazz = Class.forName(env.getProperty("userDao"));
                  userDao = (UserDao) clazz.newInstance();
              } catch (ClassNotFoundException e) {
                  e.printStackTrace();
              } catch (InstantiationException e) {
                  e.printStackTrace();
              } catch (IllegalAccessException e) {
                  e.printStackTrace();
              }
              return userDao;
          }
      }
      ```

      自定义配置文件applicationContext.properties:

      ```xml
      # 自定义properties文件，存储要管理的类全名
      # 在java代码中用对应的Properties集合 来读取这个文件的内容
      # Properties是专门处理属性文件的类，以键值对的形式存储
      # Properties [userService = com.yuziyan.xxx.UserServiceImpl]
      # Properties.getProperty("userService")
      
      userDao=com.yuziyan.basic.UserDaoImpl
      userService=com.yuziyan.basic.UserServiceImpl
      ```

   3. 通用工厂的设计

      - 通用工厂的代码：

        ```java
        package com.yuziyan.basic;
        
        import java.io.IOException;
        import java.io.InputStream;
        import java.util.Properties;
        
        public class BeanFactory {
        
            private static Properties env = new Properties();
            static{
                try {
                    //第一步 获得IO输入流
                    InputStream inputStream = BeanFactory.class.getResourceAsStream("applicationContext.properties");
                    //第二步 文件内容 封装 Properties集合中 key = userService value = com.yuziyan.UserServiceImpl
                    env.load(inputStream);
                } catch (IOException e) {
                    e.printStackTrace();
                }
        
            }
        
            //通用工厂
            public static Object getBean(String key){
                Object res = null;
                try {
                    Class clazz = Class.forName(env.getProperty(key));
                    res = clazz.newInstance();
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                } catch (InstantiationException e) {
                    e.printStackTrace();
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                return res;
            }
        }
        
        ```

      - 通用工厂的使用方式：

        ```
        1. 定义类型 (类)
        2. 通过配置文件的配置告知工厂(applicationContext.properties)
           key = value
        3. 通过工厂获得类的对象
           Object ret = BeanFactory.getBean("key")
        ```

        

5. 总结

   **Spring本质：**工厂 ApplicationContext (applicationContext.xml)

## 第二章 Spring 程序

1. Spring 的核心API

   - ApplicationContext

     ```
     作用：Spring提供的ApplicationContext这个工厂，用于对象创建
     好处：解耦合
     ```

   - 特点：

     ```
     ApplicationContext是接口类型，屏蔽了实现的差异。
     非web环境 ： ClassPathXmlApplicationContext (main junit)
     web环境  ：  XmlWebApplicationContext
     ```

   - 重量级资源

     ```
     ApplicationContext工厂的对象占用大量内存。
     不会频繁的创建对象 ： 一个应用只会创建一个工厂对象。
     ApplicationContext工厂：一定是线程安全的(多线程并发访问)
     ```

     

2. 程序开发

   ```
   1. 创建类型
   2. 配置文件的配置 applicationContext.xml
      <bean id="person" class="com.yuziyan.basic.Person"/>
   3. 通过工厂类，获得对象
      ApplicationContext
             |- ClassPathXmlApplicationContext 
      ApplicationContext ctx = new ClassPathXmlApplicationContext("/applicationContext.xml");
      Person person = (Person)ctx.getBean("person");
   ```

   

3.  细节分析

   - 名词解释

     ```
     Spring工厂创建的对象，叫做bean或者组件(component)
     ```

   - Spring工厂的相关方法

     ```java
     //这种方式获取对象，不需要强制类型转换
     Person person = ctx.getBean("person", Person.class);
     System.out.println("person = " + person);
     
     //当前Spring的配置文件中 只能有一个<bean class是Person类型
     Person person1 = ctx.getBean(Person.class);
     System.out.println("person1 = " + person1);
     
     //获取配置文件中所有bean标签的id值  person person1
     String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
     for (String beanDefinitionName : beanDefinitionNames) {
     System.out.println("beanDefinitionName = " + beanDefinitionName);
     }
     
     //根据类型获取配置文件中对应bean标签的id值
     String[] beanNamesForType = ctx.getBeanNamesForType(Person.class);
     for (String id : beanNamesForType) {
     System.out.println("id = " + id);
     }
     
     //用于判断是否存在指定id值的bean，不能判断name值
     System.out.println(ctx.containsBeanDefinition("a"));
     
     //用于判断是否存在指定id值的bean，可以判断name值
     System.out.println(ctx.containsBean("person"));
     ```

   - 配置文件中需要注意的细节

     ```
     1. 只配置class属性
     <bean  class="com.yuziyan.basic.Person"/>`
     a)上述这种配置，没有指定id，Spring会自动生成一个 id，com.yuziyan.basic.Person#0，可以使用 getBeanNamesForType() 等方法验证。
     b)应用场景：
     	如果这个bean只需要使用一次，那么就可以省略id值
     	如果这个bean会使用多次，或者被其他bean引用则需要设置id值
     
     2. name属性
     作用：用于在Spring的配置文件中，为bean对象定义别名(小名)
     
     name与id的相同点：
     	1. ctx.getBean("id") 或 ctx.getBean("name") 都可以创建对象；
     	2. <bean id="person" class="Person"/> 与 <bean name="person" class="Person"/> 等效；
     name与id的不同点：
     	1. 别名可以定义多个,但是 id 属性只能有⼀个值；
     	2. XML 的 id 属性的值，命名要求：必须以字⺟开头，可以包含 字⺟、数字、下划线、连字符；不能以特殊字符开头 /person；
     	XML 的 name 属性的值，命名没有要求，/person 可以。 但其实 XML 发展到了今天：ID属性的限制已经不存在，/person也可以。
     ```

   




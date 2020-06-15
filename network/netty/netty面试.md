# netty面试

## 基础
1. TCP、UDP的区别？

   [TCP与UDP的区别](https://mp.weixin.qq.com/s/4wccVpYf_eOf-g5l609LrA)

2. TCP协议如何保证可靠传输？
   
    [TCP 协议如何保证可靠传输](https://www.jianshu.com/p/6aac4b2a9fd7)
    
3. TCP的握手、挥手机制？

    [TCP三次握手以及四次挥手机制](https://blog.csdn.net/weixin_43349527/article/details/88130185)

    [TCP的三次握手与四次挥手](https://blog.csdn.net/qzcsu/article/details/72861891)

4. TCP的粘包/拆包原因及其解决方法是什么？

    [TCP粘包，拆包及解决方法](https://blog.csdn.net/wxy941011/article/details/80428470)

5. Netty的粘包/拆包是怎么处理的，有哪些实现？

    [TCP粘包/拆包问题和Netty的解决方案](https://www.jianshu.com/p/d65d03cb3466)

6. 同步与异步、阻塞与非阻塞的区别？

    同步：执行一个操作之后，等待结果，然后才继续执行后续的操作。
    
    异步：执行一个操作后，可以去执行其他的操作，然后等待通知再回来执行刚才没执行完的操作

    阻塞：进程给CPU传达一个任务之后，一直等待CPU处理完成，然后才执行后面的操作

    非阻塞：进程给CPU传达任务后，继续处理后续的操作，隔断时间再来询问之前的操作是否完成。这样的过程其实也叫轮询。
    
7. 说说网络IO模型？

    *[IO模块总结](https://my.oschina.net/keepal/blog/3221768/print)

8. BIO、NIO、AIO分别是什么？

    [BIO，NIO，AIO的区别](https://blog.csdn.net/u013068377/article/details/70312551)

9. select、poll、epoll的机制及其区别？

    *[IO多路复用的三种机制Select，Poll，Epoll](https://www.jianshu.com/p/397449cadc9a)
    
    [select、poll、epoll之间的区别(搜狗面试)](https://www.cnblogs.com/aspirant/p/9166944.html)

11. Netty跟Java NIO有什么不同，为什么不直接使用JDK NIO类库？

12. Netty组件有哪些，分别有什么关联？

13. 说说Netty的执行流程？


## 高级
1. Netty高性能体现在哪些方面？

2. Netty的线程模型是怎么样的？

3. Netty的零拷贝提体现在哪里，与操作系统上的有什么区别？

4. Netty的内存池是怎么实现的？

5. Netty的对象池是怎么实现的？

以上问题参考：
[Netty面试题](https://www.jianshu.com/p/a3b8efb72d04)
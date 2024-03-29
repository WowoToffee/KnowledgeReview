# HTTP三次四次挥手

本文通过图来梳理TCP-IP协议相关知识。TCP通信过程包括三个步骤：建立TCP连接通道，传输数据，断开TCP连接通道。如图1所示，给出了TCP通信过程的示意图。

![TCP通信](https://res-static.hc-cdn.cn/fms/img/0c6f5dc04e337b1c6deb57e82a9210d71603427626959)

三次握手建立连接：

第一次握手：客户端发送syn包(seq=x)到服务器，并进入SYN_SEND状态，等待服务器确认;

第二次握手：服务器收到syn包，必须确认客户的SYN(ack=x+1)，同时自己也发送一个SYN包(seq=y)，即SYN+ACK包，此时服务器进入SYN_RECV状态;

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=y+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

握手过程中传送的包里不包含数据，三次握手完毕后，客户端与服务器才正式开始传送数据。理想状态下，TCP连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP 连接都将被一直保持下去。

传输数据过程：

a.超时重传超时重传机制用来保证TCP传输的可靠性。每次发送数据包时，发送的数据报都有seq号，接收端收到数据后，会回复ack进行确认，表示某一seq 号数据已经收到。发送方在发送了某个seq包后，等待一段时间，如果没有收到对应的ack回复，就会认为报文丢失，会重传这个数据包。

b.快速重传接受数据一方发现有数据包丢掉了。就会发送ack报文告诉发送端重传丢失的报文。如果发送端连续收到标号相同的ack包，则会触发客户端的快速重 传。比较超时重传和快速重传，可以发现超时重传是发送端在傻等超时，然后触发重传;而快速重传则是接收端主动告诉发送端数据没收到，然后触发发送端重传。

c.流量控制这里主要说TCP滑动窗流量控制。TCP头里有一个字段叫Window，又叫Advertised-Window，这个字段是接收端告诉发送端自己 还有多少缓冲区可以接收数据。于是发送端就可以根据这个接收端的处理能力来发送数据，而不会导致接收端处理不过来。 滑动窗可以是提高TCP传输效率的一种机制。

d.拥塞控制滑动窗用来做流量控制。流量控制只关注发送端和接受端自身的状况，而没有考虑整个网络的通信情况。拥塞控制，则是基于整个网络来考虑的。考虑一下这 样的场景：某一时刻网络上的延时突然增加，那么，TCP对这个事做出的应对只有重传数据，但是，重传会导致网络的负担更重，于是会导致更大的延迟以及更多 的丢包，于是，这个情况就会进入恶性循环被不断地放大。试想一下，如果一个网络内有成千上万的TCP连接都这么行事，那么马上就会形成“网络风 暴”，TCP这个协议就会拖垮整个网络。为此，TCP引入了拥塞控制策略。拥塞策略算法主要包括：慢启动，拥塞避免，拥塞发生，快速恢复。

四次握手断开连接：

第一次挥手：主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不会再给你发数据了(当 然，在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发这些数据)，但此时主动关闭方还可以接受数据。

第二次挥手：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1(与SYN相同，一个FIN占用一个序号)。

第三次挥手：被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了。

第四次挥手：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。

### 1.1**握手🤝为什么需要三次呢，如果把最后一次的去掉改为两次握手🤝是否可行呢?**

假如现在客户端想向服务端进行握手，它发送了第一个连接的请求报文，但是由于网络信号差或者服务器负载过多，这个请求没有立即到达服务端，而是在某个网络节点中长时间的滞留了，以至于滞留到客户端连接释放以后的某个时间点才到达服务端，那么这就是一个失效的报文，但是服务端接收到这个失效的请求报文后，就误认为客户端又发了一次连接请求，服务端就会想向客户端发出确认的报文，表示同意建立连接

试想如果是用两次握手，则会出现下面这种情况：

> 如客户端发出连接请求，但因连接请求报文丢失而未收到确认，于是客户端再重传一次连接请求。后来收到了确认，建立了连接。数据传输完毕后，就释放了连接，客户端共发出了两个连接请求报文段，其中第一个丢失，第二个到达了服务端，但是第一个丢失的报文段只是在**某些网络结点长时间滞留了，延误到连接释放以后的某个时间才到达服务端**，此时服务端误认为客户端又发出一次新的连接请求，于是就向客户端发出确认报文段，同意建立连接，不采用三次握手，只要服务端发出确认，就建立新的连接了，此时客户端忽略服务端发来的确认，也不发送数据，则服务端一致等待客户端发送数据，浪费资源。

采用三次握手的办法就是为了防止上述这种情况的发生，比如就在刚才的情况下，客户端不会向服务端发出确认的请求，服务端会因为收不到确认的报文，就知道客户端并没有要建立连接，那么服务端也就不会去建立连接，这就是三次握手的作用。

### 1.2 什么是半连接队列？

服务器第一次收到客户端的 SYN 之后，就会处于 SYN_RCVD 状态，此时双方还没有完全建立其连接，服务器会把此种状态下请求连接放在一个**队列**里，我们把这种队列称之为**半连接队列**。

当然还有一个**全连接队列**，就是已经完成三次握手，建立起连接的就会放在全连接队列中。如果队列满了就有可能会出现丢包现象。

这里在补充一点关于**SYN-ACK 重传次数**的问题：
服务器发送完SYN-ACK包，如果未收到客户确认包，服务器进行首次重传，等待一段时间仍未收到客户确认包，进行第二次重传。如果重传次数超过系统规定的最大重传次数，系统将该连接信息从半连接队列中删除。
注意，每次重传等待的时间不一定相同，一般会是指数增长，例如间隔时间为 1s，2s，4s，8s......

### 1.3 ISN(Initial Sequence Number)是固定的吗？

当一端为建立连接而发送它的SYN时，它为连接选择一个初始序号。ISN随时间而变化，因此每个连接都将具有不同的ISN。ISN可以看作是一个32比特的计数器，每4ms加1 。这样选择序号的目的在于防止在网络中被延迟的分组在以后又被传送，而导致某个连接的一方对它做错误的解释。

**三次握手的其中一个重要功能是客户端和服务端交换 ISN(Initial Sequence Number)，以便让对方知道接下来接收数据的时候如何按序列号组装数据。如果 ISN 是固定的，攻击者很容易猜出后续的确认号，因此 ISN 是动态生成的。**

### 1.4 三次握手过程中可以携带数据吗？

其实第三次握手的时候，是可以携带数据的。但是，**第一次、第二次握手不可以携带数据**

为什么这样呢?大家可以想一个问题，假如第一次握手可以携带数据的话，如果有人要恶意攻击服务器，那他每次都在第一次握手中的 SYN 报文中放入大量的数据。因为攻击者根本就不理服务器的接收、发送能力是否正常，然后疯狂着重复发 SYN 报文的话，这会让服务器花费很多时间、内存空间来接收这些报文。

也就是说，**第一次握手不可以放数据，其中一个简单的原因就是会让服务器更加容易受到攻击了。而对于第三次的话，此时客户端已经处于 ESTABLISHED 状态。对于客户端来说，他已经建立起连接了，并且也已经知道服务器的接收、发送能力是正常的了，所以能携带数据也没啥毛病。**

### 1.5 SYN攻击是什么？

**服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的**，所以服务器容易受到SYN洪泛攻击。SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，由于源地址不存在，因此Server需要不断重发直至超时，这些伪造的SYN包将长时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。SYN 攻击是一种典型的 DoS/DDoS 攻击。

检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。在 Linux/Unix 上可以使用系统自带的 netstats 命令来检测 SYN 攻击。

```1c
netstat -n -p TCP | grep SYN_RECV
```

常见的防御 SYN 攻击的方法有如下几种：

- 缩短超时（SYN Timeout）时间
- 增加最大半连接数
- 过滤网关防护
- SYN cookies技术

### **2.1为什么断开连接需要四次挥手👋呢，像建立连接的时候一样，三次行不行呢？**

`TCP`协议是一种面向连接的、可靠的、基于字节流的运输层通信协议。`TCP`是全双工 模式，这就意味着，在客户端想要断开连接时，客户端向服务端发送`FIN`报文，只是表示客户端已经没有数据要发送了，但是这个时候客户端还是可以接收来自服务端的数据。

当服务端接收到`FIN`报文，并返回`ACK`报文，表示服务端已经知道了客户端要断开连接，客户端已经没有数据要发送了，但是这个时候服务端可能依然有数据要传输给客户端。

当服务端的数据传输完之后，服务端会发送`FIN`报文给客户端，表示服务端也没有数据要传输了，服务端同意关闭连接，之后，客户端收到`FIN`报文，立即发送给客户端一个`ACK`报文，确定关闭连接。在之后，客户端和服务端彼此就愉快的断开了这次的`TCP`连接。

或许会有疑问，为什么服务端的`ACK`报文和`FIN`报文都是分开发送的，但是在三次握手的时候却是`ACK`报文和`SYN`报文是一起发送的，因为在三次握手的过程中，当服务端收到客户端的`SYN`连接请求报文后，可以直接发送`SYN+ACK`报文。其中`ACK`报文是用来应答的，`SYN`报文是用来同步的。但是在关闭连接时，当服务端接收到`FIN`报文时，很可能并不会立即关闭`SOCKET`，所以只能先回复一个`ACK`报文，告诉客户端，你发的`FIN`报文我收到了，只有等到服务端所有的数据都发送完了，才能发送`FIN`报文，因此`ACK`报文和`FIN`报文不能一起发送。所以断开连接的时候才需要四次挥手来完成。

### 2.2 2MSL等待状态

TIME_WAIT状态也成为2MSL等待状态。每个具体TCP实现必须选择一个报文段最大生存时间MSL（Maximum Segment Lifetime），它是任何报文段被丢弃前在网络内的最长时间。这个时间是有限的，因为TCP报文段以IP数据报在网络内传输，而IP数据报则有限制其生存时间的TTL字段。

对一个具体实现所给定的MSL值，处理的原则是：当TCP执行一个主动关闭，并发回最后一个ACK，该连接必须在TIME_WAIT状态停留的时间为2倍的MSL。这样可让TCP再次发送最后的ACK以防这个ACK丢失（另一端超时并重发最后的FIN）。

这种2MSL等待的另一个结果是这个TCP连接在2MSL等待期间，定义这个连接的插口（客户的IP地址和端口号，服务器的IP地址和端口号）不能再被使用。这个连接只能在2MSL结束后才能再被使用。

### 2.3 四次挥手释放连接时，等待2MSL的意义?

> **MSL**是Maximum Segment Lifetime的英文缩写，可译为“最长报文段寿命”，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。

为了保证客户端发送的最后一个ACK报文段能够到达服务器。因为这个ACK有可能丢失，从而导致处在LAST-ACK状态的服务器收不到对FIN-ACK的确认报文。服务器会超时重传这个FIN-ACK，接着客户端再重传一次确认，重新启动时间等待计时器。最后客户端和服务器都能正常的关闭。假设客户端不等待2MSL，而是在发送完ACK之后直接释放关闭，一但这个ACK丢失的话，服务器就无法正常的进入关闭连接状态。

#### 两个理由：

1. 保证客户端发送的最后一个ACK报文段能够到达服务端。
   这个ACK报文段有可能丢失，使得处于LAST-ACK状态的B收不到对已发送的FIN+ACK报文段的确认，服务端超时重传FIN+ACK报文段，而客户端能在2MSL时间内收到这个重传的FIN+ACK报文段，接着客户端重传一次确认，重新启动2MSL计时器，最后客户端和服务端都进入到CLOSED状态，若客户端在TIME-WAIT状态不等待一段时间，而是发送完ACK报文段后立即释放连接，则无法收到服务端重传的FIN+ACK报文段，所以不会再发送一次确认报文段，则服务端无法正常进入到CLOSED状态。
2. 防止“已失效的连接请求报文段”出现在本连接中。
   客户端在发送完最后一个ACK报文段后，再经过2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失，使下一个新的连接中不会出现这种旧的连接请求报文段。

### 2.4 为什么TIME_WAIT状态需要经过2MSL才能返回到CLOSE状态？

理论上，四个报文都发送完毕，就可以直接进入CLOSE状态了，但是可能网络是不可靠的，有可能最后一个ACK丢失。所以**TIME_WAIT状态就是用来重发可能丢失的ACK报文**。

### 2.5 NoHttpResponseException异常

处理方式：

```
（1）http请求使用重发机制，捕获NohttpResponseException的异常，重新发送请求，重发3次后还是失败才停止。由于不知道客户端捕获到NohttpResponseException这个异常后，客户端是否自动关闭了这个连接，每次重发都需要新建连接发送。新建连接不存在太长的空闲时间问题，因此能够通过重发解决交易失败的问题。
（2）设置Keep-Alive模式，我方系统主动检查每个连接的空闲时间，允许设置连接的最大空闲时间M，即客户端建立的连接空闲M秒后，自动发起断开连接。只要这个M时间小于服务端的最大空闲时间，将完全避免服务端主动断开连接导致的异常。

```

什么是**Keep-Alive模式**？

HTTP协议采用请求-应答模式，有普通的非KeepAlive模式，也有KeepAlive模式。

非KeepAlive模式时，每个请求/应答客户和服务器都要新建一个连接，完成 之后立即断开连接（HTTP协议为无连接的协议）；当使用Keep-Alive模式（又称持久连接、连接重用）时，Keep-Alive功能使客户端到服 务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。

（1）http 1.0中默认是关闭的，如果客户端浏览器支持Keep-Alive，那么就在HTTP请求头中添加一个字段 **Connection: Keep-Alive**，当服务器收到附带有Connection: Keep-Alive的请求时，它也会在响应头中添加一个同样的字段来使用Keep-Alive。这样一来，客户端和服务器之间的HTTP连接就会被保持，不会断开（超过Keep-Alive规定的时间，意外断电等情况除外），当客户端发送另外一个请求时，就使用这条已经建立的连接。

也有可能是Keep-Alive

```text
Keep-Alive: timeout=5, max=100
timeout：过期时间5秒（对应httpd.conf里的参数是：KeepAliveTimeout），max是最多一百次请求，强制断掉连接
```



（2）http 1.1中默认启用Keep-Alive， 默认情况下所在HTTP1.1中所有连接都被保持，除非在请求头或响应头中指明要关闭：Connection: Close ，这也就是为什么Connection: Keep-Alive字段再没有意义的原因。另外，还添加了一个新的字段Keep-Alive:，因为这个字段并没有详细描述用来做什么，可忽略它



目前大部分浏览器都是用http1.1协议，也就是说默认都会发起Keep-Alive的连接请求了，所以是否能完成一个完整的Keep- Alive连接就看服务器设置情况。

3、启用Keep-Alive的优点

从上面的分析来看，启用Keep-Alive模式肯定更高效，性能更高。因为避免了建立/释放连接的开销。
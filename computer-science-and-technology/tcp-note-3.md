## 开发视角的TCP

> https://zhuanlan.zhihu.com/p/141468204

### 握手

- 发送方生成一个随机数‘X’发用一个SYN包到接收方
- 接收方对‘X’加一，生成一随机数‘Y’，返回一个SYN/ACK包
- 发送方增加‘X’、‘Y’，应答一个ACK包和应用数据。
- ‘X’、‘Y’用于保证TCP数据包的序列，确保没有间断

### 流控

流控制是一种回退机制，用于防止发送方压倒接收方。

接收方将等待应用程序处理的传入TCP数据包存储到接收缓冲区中。

无论何时接收方接收到数据包，它都会将缓冲区的大小反馈给发送方。发送方，如果它尊重协议，避免发送更多的数据，可以容纳在接收方的缓冲区。

这种机制与应用程序级别的速度限制并没有太大的不同。但是，TCP在连接级别上限制速率，而不是在API或IP地址上限制速率。 发送方和接收方之间的往返时间(RTT)越低，发送方就越快地根据接收方的容量调整其输出带宽。

### 拥塞控制

TCP不仅可以保护接收方，还可以保护底层网络。 发送方如何知道底层网络的可用带宽是多少?估计它的唯一方法是通过实际测量。

其思想是发送方维护一个所谓的“拥塞窗口”。“该窗口表示未完成的数据包的总数，可以发送而无需等待来自另一方的确认。”接收器窗口的大小限制了拥塞窗口的最大大小。拥塞窗口越小，在任何给定时间可以传输的字节就越少，使用的带宽也就越少。

当建立新连接时，拥塞窗口的大小被设置为系统默认值。然后，对于每一个被确认的包，窗口的大小会呈指数级增长。这意味着我们不能在连接建立后立即使用网络的全部容量。同样，往返时间越短，发送方就越快开始利用底层网络的带宽。

如果丢了包怎么办?当发送方通过超时检测到错过的确认时，一种称为“拥塞避免”的机制就会启动，拥塞窗口大小就会减小。从那时起，时间将使窗口大小增加一定的数量，而超时将使窗口大小减少另一个数量。

如前所述，拥塞窗口的大小定义了无需等待确认即可发送的最大比特数。发送方需要等待一个完整的往返过程才能获得确认。因此，通过将拥塞窗口的大小除以往返时间，我们可以得到最大的理论带宽:

Bandwidth=WinSize/RTT

这个简单的等式表明带宽是延迟的函数。TCP会非常努力地优化窗口大小，因为它对往返时间无能为力。但这并不总能得到最优的配置。

总而言之，拥塞控制是一种自适应机制，用于推断网络的底层带宽和拥塞。在应用程序级别也可以应用类似的模式。想象一下你在Netflix上看电影时会发生什么。它开始模糊的;然后，它稳定到一个合理的水平，直到出现一个小问题，质量再次恶化。这种应用于视频流的机制称为自适应比特率流。

## 握手挥手其他参考

> https://blog.csdn.net/qzcsu/article/details/72861891

### 握手

- TCP服务器进程先创建传输控制块TCB，时刻准备接受客户进程的连接请求，此时服务器就进入了LISTEN（监听）状态；
- TCP客户进程也是先创建传输控制块TCB，然后向服务器发出连接请求报文，这是报文首部中的同部位SYN=1，同时选择一个初始序列号 seq=x ，此时，TCP客户端进程进入了 SYN-SENT（同步已发送状态）状态。TCP规定，SYN报文段（SYN=1的报文段）不能携带数据，但需要消耗掉一个序号。
- TCP服务器收到请求报文后，如果同意连接，则发出确认报文。确认报文中应该 ACK=1，SYN=1，确认号是ack=x+1，同时也要为自己初始化一个序列号 seq=y，此时，TCP服务器进程进入了SYN-RCVD（同步收到）状态。这个报文也不能携带数据，但是同样要消耗一个序号。
- TCP客户进程收到确认后，还要向服务器给出确认。确认报文的ACK=1，ack=y+1，自己的序列号seq=x+1，此时，TCP连接建立，客户端进入ESTABLISHED（已建立连接）状态。TCP规定，ACK报文段可以携带数据，但是如果不携带数据则不消耗序号。
- 当服务器收到客户端的确认后也进入ESTABLISHED状态，此后双方就可以开始通信了。

### 挥手

- 客户端进程发出连接释放报文，并且停止发送数据。释放数据报文首部，FIN=1，其序列号为seq=u（等于前面已经传送过来的数据的最后一个字节的序号加1），此时，客户端进入FIN-WAIT-1（终止等待1）状态。 TCP规定，FIN报文段即使不携带数据，也要消耗一个序号。
- 服务器收到连接释放报文，发出确认报文，ACK=1，ack=u+1，并且带上自己的序列号seq=v，此时，服务端就进入了CLOSE-WAIT（关闭等待）状态。TCP服务器通知高层的应用进程，客户端向服务器的方向就释放了，这时候处于半关闭状态，即客户端已经没有数据要发送了，但是服务器若发送数据，客户端依然要接受。这个状态还要持续一段时间，也就是整个CLOSE-WAIT状态持续的时间。
- 客户端收到服务器的确认请求后，此时，客户端就进入FIN-WAIT-2（终止等待2）状态，等待服务器发送连接释放报文（在这之前还需要接受服务器发送的最后的数据）。
- 服务器将最后的数据发送完毕后，就向客户端发送连接释放报文，FIN=1，ack=u+1，由于在半关闭状态，服务器很可能又发送了一些数据，假定此时的序列号为seq=w，此时，服务器就进入了LAST-ACK（最后确认）状态，等待客户端的确认。
- 客户端收到服务器的连接释放报文后，必须发出确认，ACK=1，ack=w+1，而自己的序列号是seq=u+1，此时，客户端就进入了TIME-WAIT（时间等待）状态。注意此时TCP连接还没有释放，必须经过2∗ *∗MSL（最长报文段寿命）的时间后，当客户端撤销相应的TCB后，才进入CLOSED状态。
- 服务器只要收到了客户端发出的确认，立即进入CLOSED状态。同样，撤销TCB后，就结束了这次的TCP连接。可以看到，服务器结束TCP连接的时间要比客户端早一些。

## TCP状态机

https://www.jianshu.com/p/3c7a0771b67e

### 状态

状态|	描述
--|--
LISTEN	|等待来自远程TCP应用程序的请求
SYN_SENT|	发送连接请求后等待来自远程端点的确认。TCP第一次握手后客户端所处的状态
SYN-RECEIVED|	该端点已经接收到连接请求并发送确认。该端点正在等待最终确认。TCP第二次握手后服务端所处的状态
ESTABLISHED|	代表连接已经建立起来了。这是连接数据传输阶段的正常状态
FIN_WAIT_1|	等待来自远程TCP的终止连接请求或终止请求的确认
FIN_WAIT_2|	在此端点发送终止连接请求后，等待来自远程TCP的连接终止请求
CLOSE_WAIT|	该端点已经收到来自远程端点的关闭请求，此TCP正在等待本地应用程序的连接终止请求
CLOSING|	等待来自远程TCP的连接终止请求确认
LAST_ACK|	等待先前发送到远程TCP的连接终止请求的确认
TIME_WAIT|	等待足够的时间来确保远程TCP接收到其连接终止请求的确认

###  TIME_WAIT

TIME_WAIT状态应该是最让人疑惑的一个状态了。在上图中可以看到，执行主动断开的节点最后会进入这个状态，该节点会在此状态保存2倍的MSL（最大段生存期）。

TCP的每个实现都必须为MSL选择一个值。RFC 1122推荐的值为两分钟，伯克利派的实现使用30秒。这也就是说TIME_WAIT状态会维持1到4分钟。MSL是任何IP数据报可以在网络中生存的最长时间。这个时间是有限制的，因为每个数据报都包含一个8位的跳数限制，最大值是255.虽然这是一个跳数限制而不是一个真正的时间限制，但是根据这个限制来假设数据报的最长生命周期依然是有意义的。

网络中数据报丢失的原因通常是路由异常。一旦路由崩溃或者两个路由之间的链路断开，路由协议需要几秒或几分钟才能稳定，并找到一条备用路径。在这段时间内，可能发生路由回路。同时假设丢失是一个TCP数据报，则发生TCP超时，并且重新发送分组，重传的分组通过一些备用路径达到最终目的地。但是一段时间后（该时间小于MSL），路由循环被更正，在循环中丢失的数据报被发送到最终目的地。这个原始的数据报被称为丢失的副本或漫游副本。TCP协议必须处理这些数据报。

维持TIME_WAIT有两个原因：

可靠地实现TCP的全双工连接终止。
允许旧的重复数据段在网络中过期
在四次挥手中，假设最后的ACK丢失了，被动关闭方会重发FIN。主动关闭端必须维护状态，来允许被动关闭方重发最后的ACK；如果它没有维护这个状态，将会对重发FIN返回RST，被动关闭方会认为这是个错误。如果TCP正在执行彻底终止数据流的两个方向所需的所有工作（即全双工关闭），则必须正确处理这四个段中任何一个的丢失。所以执行主动关闭的一方必须在结束时保持TIME_WAIT状态：因为它可能必须重传最后的ACK。

现在来聊维持TIME_WAIT状态的第二个原因。假设在主机12.106.32.254的1500端口和206.168.112.219的21端口之间有一个TCP连接。此连接关闭后，在相通的地址和端口建立了另外一个连接。由于IP地址和端口相同，所以后一种连接被称为先前连接的“化身”。TCP必须防止连接中的旧副本在稍后再次出现，并被误解为属于同一连接的新“化身”。为此，TCP将不会启动当前处于TIME_WAIT状态的连接的新“化身”。由于TIME_WAIT状态的持续时间时两倍的MSL，因此TCP允许一个方向的数据在MSL秒内丢失，也允许回复在一个MSL秒内丢失。通过强制执行此规则，可以保证当一个TCP连接成功建立时，来自先前连接的所有旧的副本在网络中已过期。
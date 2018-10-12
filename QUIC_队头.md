前一阵子给团队进行了一次QUIC协议的介绍，我们说到QUIC的一个特性就是能够解决队头阻塞的问题。但["队头堵塞"](https://zh.wikipedia.org/wiki/%E9%98%9F%E5%A4%B4%E9%98%BB%E5%A1%9E)（Head-of-line blocking） ，其实主要说的是两个方面的队头阻塞：

1. http request级别的队头阻塞问题 
2. 传输层的队头阻塞问题 

那QUIC协议所做的避免队头阻塞，究竟做的是哪方面的事情？



## **如何解决http request级别的队头阻塞**

要想理解 QUIC 协议到底做了什么以及这么做的必要性，我想还是从最基础的 HTTP 1.0 聊起比较合适。 

#### **管道机制(Pipelining)**

根据谷歌的调查， 现在请求一个网页，平均涉及到 80 个资源，30 多个域名。考虑最原始的情况，每请求一个资源都需要建立一次 TCP 请求。

1999 年提出的 [HTTP 1.1 协议](https://www.ietf.org/rfc/rfc2616.txt) 中就把 `Connection` 的默认值改成了`Keep-Alive`，这样同一个域名下的多个 HTTP 请求就可以复用同一个 TCP 连接。这种做法被称为 HTTP Pipeline，优点是显著的减少了建立连接的次数，也就是大幅度减少了 RTT。以上面的数据为例，如果 80 个资源都要走一次 HTTP 1.0，那么需要建立 80 个 TCP 连接，握手 80 次，也就是 80 个 RTT。如果采用了 HTTP 1.1 的 Pipeline，只需要建立 30 个 TCP 连接，也就是 30 个 RTT，提高了 **62.5%** 的效率。

Pipeline 解决了 TCP 连接浪费的问题，但它自己还存在一些不足之处，也就是所有管道模型都难以避免的队头阻塞问题。

#### http request级别的队头阻塞

**http request级别中的队头阻塞说的复用同一个TCP连接期间，即便是通过管道同时发送了多个请求，服务端也是按请求的顺序依次给出响应的；而客户端在未收到之前所发出所有请求的响应之前，将会阻塞后面的请求(排队等待)。**
![1.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/c5b5750bb04a234fafdbb5555ed0ff3a.png)

比如上图中，如果第四个资源的传输花了很久，后面的资源都得等着，平白浪费了很多时间，带宽资源没有得到充分利用。

因此，HTTP 协议允许客户端发起多个并行请求，比如在笔者的机器上Chrome最多支持六个并发请求。并发请求主要是用于解决 队头阻塞问题，当有三个并发请求时，情况会变成这样:
![2.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/3c1da88535d525be72add01d1de2ca4c.png)
可见虽然第四个资源的请求被阻塞了，但是其他的资源请求并不一定会被阻塞，这样总的来说网络的平均利用率得到了提升。

所以，并发请求并非是直接解决了 队头阻塞的问题，而是尽可能减少 队头阻塞造成的影响”， 

#### **多路复用(Multiplexing)**

------

虽然 HTTP 1.1 默认启用长TCP连接，但所有的请求-响应都是按序进行的。

`HTTP/2`是怎么解决这个问题的呢？那就是采用`二进制数据分帧`和`流`，其中`二进制数据分帧`对数据进行顺序标识，这样浏览器收到数据之后，就可以按照序列对数据进行合并。同样是因为有了序列，服务器就可以并行传输数据，并发进行多个`流`的传输。

多个请求复用一个TCP连接，然后每个`request-response`都被拆分为若干个frame发送，这样即使一个请求被阻塞了，也不会影响其他请求。



那么多路复用已经够厉害了，它解决了队头阻塞问题？很遗憾的是，并没有。实际上，只要你还在用 TCP 链接，队头阻塞就是逃不掉的噩梦，我们来看看 TCP 的实现细节。



##  传输层的队头阻塞问题 

我们知道 TCP 协议会保证数据的可达性，如果发生了丢包或者错包，数据就会被重传。于是问题来了，如果一个包丢了，那么后面的包就得停下来等这个包重新传输，也就是发生了队头阻塞。当然 TCP 协议的设计者们也不傻，他们发明了滑动窗口的概念:
![3.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/66aaaaff8a9a36d9f1423dc757a69eaf.png)

这样的好处是在第一个数据包(1-1000) 发出后，不必等到 ACK 返回就可以立刻发送第二个数据包。可以看出图中的 TCP 窗口大小是 4，所以第四个包发送后就会开始等待，直到第一个包的 ACK 返回。这样窗口可以向后滑动一位，第五个包被发送。

如果第一、二、三个的包都丢失了也没有关系，当接收方收到第四个包时，它可以确信一定是前三个 ACK 丢了而不是数据包丢了，否则不会收到 4001 的 ACK，所以发送方可以大胆的把窗口向后滑动四位。

滑动窗口的概念大幅度提高了 TCP 传输数据时抗干扰的能力，一般丢失一两个 ACK 根本没关系。但如果是发送的包丢失，或者出错，窗口就无法向前滑动，也会出现队头阻塞的现象。

因此，如果队头阻塞的粒度是`http request`这个级别，那么`HTTP/2 over TCP`的确解决了`HTTP/1.1`中的问题。但是，`HTTP/2`目前实现层面上都是基于TCP，因此`HTTP/2`并没有解决数据传输层的队头阻塞问题。 



## QUIC如何解决传输层的队头阻塞

应用层无法解决传输层的问题。而目前我们所说的传输层的队头阻塞，主要指的是TCP中的队头阻塞，而要重新设计和实现TCP，由于协议历史悠久导致中间设备僵化 ，而协议本身强依赖于操作系统的实现，导致协议本身僵化。因此修改TCP产生的迭代周期非常长，也很难推进。而使用TCP协议进行传输，又会存在队头阻塞的问题。那么如何解决这个问题。Google的QUIC把传输层使用的协议瞄向了UDP。


![4.png](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/5df23fa080a1a0fe84f98a1340745052.png)

从上图可以看出，QUIC底层通过UDP协议替代了TCP。

一方面，QUIC的实现包含了多路复用，解决了http request级别的队头阻塞。另一方面，QUIC 协议基于 UDP 实现，我们知道UDP是面向数据报文的，数据包之间没有阻塞约束，也没有严格的顺序，当一个数据包遇到问题需要重传时，只会影响该数据包对应的资源，其他独立的资源（如其他css、js文件）不会受到影响。QUIC就是充分利用这个特性解决传输层的队头阻塞问题的。 

需要说明的是，当前的QUIC实现使用[HPACK压缩http header](https://docs.google.com/document/d/1WJvyZflAO2pq77yOLbp9NsGjC1CHetAXV8I0fQe-B_U/edit#heading=h.xkjc5yjm9sgx), 受限于当前HPACK算法实现，在QUIC中的header帧也是受队头阻塞的。但是粒度已经降低到了帧这个级别，并且仅会在header帧中出现。实际使用中，出现的概率已经非常低了。 

当然，QUIC的协议实现有非常多的细节，如果你想进一步了解，可以关注他们的[开源实现](https://chromium.googlesource.com/chromium/src/+/master/net/quic/)。 

## 小结

1. `HTTP/2 over TCP`以及`QUIC over UDP`中的多路复用解决了http request级别的队头阻塞问题
2. `QUIC over UDP`解决了传输层的队头阻塞问题（除去header frame）

## 参考资料

[1]: https://docs.google.com/document/d/1g5nIXAIkN_Y-7XJW5K45IblHd_L2f5LTaDUDwvZ5L6g/edit	"QUIC crypto design doc"
[2]: http://www.52im.net/thread-1309-1-1.html	"新一代基于UDP的低延时网络传输层协议——QUIC详解"
[3]: https://zhuanlan.zhihu.com/p/32553477	"科普:QUIC协议原理分析"
[4]: https://www.slideshare.net/obonaventure/beyond-tcp-the-evolution-of-internet-transport-protocols	"Beyond TCP: The evolution of Internet transport protocols"
[5]: https://liudanking.com/arch/what-is-head-of-line-blocking-http2-quic/	"当我们在谈论HTTP队头阻塞时，我们在谈论什么？"
[6]: https://xiaozhuanlan.com/topic/2083674195	"试图取代 TCP 的 QUIC 协议到底是什么?"
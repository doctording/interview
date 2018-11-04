<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [tcp udp区别](#tcp-udp区别)
    - [各自优缺点](#各自优缺点)
    - [TCP UDO什么时候使用](#tcp-udo什么时候使用)
- [TCP三次握手及四次挥手](#tcp三次握手及四次挥手)
    - [三次握手的过程](#三次握手的过程)
    - [四次挥手过程（关闭客户端到服务器的连接）](#四次挥手过程关闭客户端到服务器的连接)
    - [为什么需要三次握手，两次不可以吗？或者四次、五次可以吗？](#为什么需要三次握手两次不可以吗或者四次五次可以吗)
    - [为什么需要2MSL时间？](#为什么需要2msl时间)
    - [为什么是四次挥手，而不是三次或是五次、六次？](#为什么是四次挥手而不是三次或是五次六次)
    - [如果已经建立了连接，但是客户端突然出现故障了怎么办](#如果已经建立了连接但是客户端突然出现故障了怎么办)
- [https和http的区别](#https和http的区别)
- [linux nginx配置https](#linux-nginx配置https)
- [GET POST区别](#get-post区别)
- [HTTP 状态码](#http-状态码)
    - [1XX 信息](#1xx-信息)
    - [2XX 成功](#2xx-成功)
    - [3XX 重定向](#3xx-重定向)
    - [4XX 客户端错误](#4xx-客户端错误)
    - [5XX 服务器错误](#5xx-服务器错误)

<!-- /TOC -->


# tcp udp区别

## 各自优缺点
* TCP的优点： 可靠，稳定 TCP的可靠体现在TCP在传递数据之前，会有三次握手来建立连接，而且在数据传递时，有确认、窗口、重传、拥塞控制机制，在数据传完后，还会断开连接用来节约系统资源。 
* TCP的缺点： 慢，效率低，占用系统资源高，易被攻击 TCP在传递数据之前，要先建连接，这会消耗时间，而且在数据传递时，确认机制、重传机制、拥塞控制机制等都会消耗大量的时间，而且要在每台设备上维护所有的传输连接，事实上，每个连接都会占用系统的CPU、内存等硬件资源。 而且，因为TCP有确认机制、三次握手机制，这些也导致TCP容易被人利用，实现DOS、DDOS、CC等攻击。

* UDP的优点： 快，比TCP稍安全 UDP没有TCP的握手、确认、窗口、重传、拥塞控制等机制，UDP是一个无状态的传输协议，所以它在传递数据时非常快。没有TCP的这些机制，UDP较TCP被攻击者利用的漏洞就要少一些。但UDP也是无法避免攻击的，比如：UDP Flood攻击…… 
* UDP的缺点： 不可靠，不稳定 因为UDP没有TCP那些可靠的机制，在数据传递时，如果网络质量不好，就会很容易丢包。

## TCP UDO什么时候使用
 * 什么时候应该使用TCP： 当对网络通讯质量有要求的时候，比如：整个数据要准确无误的传递给对方，这往往用于一些要求可靠的应用，比如HTTP、HTTPS、FTP等传输文件的协议，POP、SMTP等邮件传输的协议。 在日常生活中，常见使用TCP协议的应用如下： 浏览器，用的HTTP FlashFXP，用的FTP Outlook，用的POP、SMTP Putty，用的Telnet、SSH QQ文件传输 ………… 
 
 * 什么时候应该使用UDP： 当对网络通讯质量要求不高的时候，要求网络通讯速度能尽量的快，这时就可以使用UDP。 比如，日常生活中，常见使用UDP协议的应用如下： QQ语音 QQ视频 TFTP ……有些应用场景对可靠性要求不高会用到UPD，比如长视频，要求速率
 
 参考 1 :[TCP和UDP的优缺点及区别](https://www.cnblogs.com/xiaomayizoe/p/5258754.html)

 参考 2 :[TCP和UDP的区别和优缺点]( https://blog.csdn.net/xiaobangkuaipao/article/details/76793702)

[toTop](#jump)

# TCP三次握手及四次挥手

![](/img/tcp_handshake.png)

三次握手和四次挥手中的用到序列号、确认号及标志位:
1) 序列号seq
占4个字节，用来标记数据段的顺序，TCP把连接中发送的所有数据字节都编上一个序号，第一个字节的编号由本地随机产生，给字节编上序号后，就给每一个报文段指派一个序号，序列号seq就是这个报文段中的第一个字节的数据编号。

2) 确认号ack
占4个字节，期待收到对方下一个报文段的第一个数据字节的序号，序列号表示报文段携带数据的第一个字节的编号，而确认号指的是期望接受到下一个字节的编号，因此挡墙报文段最后一个字节的编号+1即是确认号。

3) 确认ACK
占1个比特位，仅当ACK=1，确认号字段才有效。ACK=0，确认号无效。

4) 同步SYN
连接建立时用于同步序号。当SYN=1，ACK=0表示：这是一个连接请求报文段。若同意连接，则在响应报文段中使用SYN=1，ACK=1.因此，SYN=1表示这是一个连接请求，或连接接收报文，SYN这个标志位只有在TCP建立连接才会被置为1，握手完成后SYN标志位被置为0.

5) 终止FIN
用来释放一个 


## 三次握手的过程
1) 第一次握手
建立连接时，客户端发送SYN包到服务器，其中包含客户端的初始序号seq=x，并进入SYN_SENT状态，等待服务器确认。（其中，SYN=1，ACK=0，表示这是一个TCP连接请求数据报文；序号seq=x，表明传输数据时的第一个数据字节的序号是x）。

2) 第二次握手
服务器收到请求后，必须确认客户的数据包。同时自己也发送一个SYN包，即SYN+ACK包，此时服务器进入SYN_RECV状态。（其中确认报文段中，标识位SYN=1，ACK=1，表示这是一个TCP连接响应数据报文，并含服务端的初始序号seq(服务器)=y，以及服务器对客户端初始序号的确认号ack(服务器)=seq(客户端)+1=x+1）。

3) 第三次握手
客户端收到服务器的SYN+ACK包，向服务器发送一个序列号(seq=x+1)，确认号为ack(客户端)=y+1，此包发送完毕，客户端和服务器进入ESTAB_LISHED(TCP连接成功)状态，完成三次握手。
未连接队列

在三次握手协议中，服务器维护一个未连接队列，该队列为每个客户端的SYN包(syn=j)开设一个条目，该条目表明服务器已收到SYN包，并向客户发出确认，正在等待客户的确认包时，删除该条目，服务器进入ESTAB_LISHED状态。

## 四次挥手过程（关闭客户端到服务器的连接）

1) 第一次挥手
首先，客户端发送一个FIN，用来关闭客户端到服务器的数据传送，然后等待服务器的确认。其中终止标志位FIN=1，序列号seq=u。

2) 第二次挥手
服务器收到这个FIN，它发送一个ACK，确认ack为收到的序号加一。

3) 第三次挥手
关闭服务器到客户端的连接，发送一个FIN给客户端。

4) 第四次挥手
客户端收到FIN后，并发回一个ACK报文确认，并将确认序号seq设置为收到序号加一。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

客户端发送FIN后，进入终止等待状态，服务器收到客户端连接释放报文段后，就立即给客户端发送确认，服务器就进入CLOSE_WAIT状态，此时TCP服务器进程就通知高层应用进程，因而从客户端到服务器的连接就释放了。此时是“半关闭状态”，即客户端不可以发送给服务器，服务器可以发送给客户端。
此时，如果服务器没有数据报发送给客户端，其应用程序就通知TCP释放连接，然后发送给客户端连接释放数据报，并等待确认。客户端发送确认后，进入TIME_WAIT状态，但是此时TCP连接还没有释放，然后经过等待计时器设置的2MSL后，才进入到CLOSE状态。

## 为什么需要三次握手，两次不可以吗？或者四次、五次可以吗？
我们来分析一种特殊情况，假设客户端请求建立连接，发给服务器SYN包等待服务器确认，服务器收到确认后，如果是两次握手，假设服务器给客户端在第二次握手时发送数据，数据从服务器发出，服务器认为连接已经建立，但在发送数据的过程中数据丢失，客户端认为连接没有建立，会进行重传。假设每次发送的数据一直在丢失，客户端一直SYN，服务器就会产生多个无效连接，占用资源，这个时候服务器可能会挂掉。这个现象就是我们听过的“SYN的洪水攻击”。
总结：第三次握手是为了防止：如果客户端迟迟没有收到服务器返回确认报文，这时会放弃连接，重新启动一条连接请求，但问题是：服务器不知道客户端没有收到，所以他会收到两个连接，浪费连接开销。如果每次都是这样，就会浪费多个连接开销。

## 为什么需要2MSL时间？
首先，MSL即Maximum Segment Lifetime，就是 **最大报文生存时间**，是任何报文在网络上的存在的最长时间，超过这个时间报文将被丢弃。《TCP/IP详解》中是这样描述的：**MSL是任何报文段被丢弃前在网络内的最长时间**。RFC 793中规定MSL为2分钟，实际应用中常用的是30秒、1分钟、2分钟等。

TCP的TIME_WAIT需要等待2MSL，当TCP的一端发起主动关闭，三次挥手完成后发送第四次挥手的ACK包后就进入这个状态，等待2MSL时间主要目的是：防止最后一个ACK包对方没有收到，那么对方在超时后将重发第三次握手的FIN包，主动关闭端接到重发的FIN包后可以再发一个ACK应答包。在TIME_WAIT状态时两端的端口不能使用，要等到2MSL时间结束才可以继续使用。当连接处于2MSL等待阶段时任何迟到的报文段都将被丢弃。

## 为什么是四次挥手，而不是三次或是五次、六次？
双方关闭连接要经过双方都同意。所以，首先是客服端给服务器发送FIN，要求关闭连接，服务器收到后会发送一个ACK进行确认。服务器然后再发送一个FIN，客户端发送ACK确认，并进入TIME_WAIT状态。等待2MSL后自动关闭。

## 如果已经建立了连接，但是客户端突然出现故障了怎么办
TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75分钟发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接

参考 1 :[TCP的三次握手与四次挥手（详解+动图）](https://blog.csdn.net/qzcsu/article/details/72861891)

参考 2 :[TCP三次握手及四次挥手详解及常见面试题](https://blog.csdn.net/ZWE7616175/article/details/80432486)


[toTop](#jump)


# https和http的区别

1) HTTPS更安全：HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比HTTP协议的信息明文传输安全；

2) HTTPS需要申请证书：HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费，费用大概与.com域名差不多，每年需要大约几十元的费用。而常见的HTTP协议则没有这一项；

3) 端口不同：HTTP使用的是大家最常见的80端口，而HTTPS连接使用的是443端口；

4) 安全性不同：HTTP的连接很简单，是无状态的。而HTTPS协议是SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比HTTP协议安全；

HTTPS跟HTTP一样，只不过增加了SSL。

1、HTTP包含如下动作：
```

    （1）浏览器打开一个TCP连接

    （2）浏览器发送HTTP请求到服务器端

    （3）服务器发送HTTP回应信息到浏览器

    （4）TCP连接关闭
```

2、SSL包含如下动作：

```

    （1）验证服务器端；

    （2）允许客户端和服务器端选择加密算法和密码，确保双方都支持

    （3）验证客户端（可选）

    （4）使用公钥加密技术来生成共享加密数据

    （5）创建一个加密的SSL连接

    （6）基于该SSL连接传递HTTP请求
```

参考1 : [https和http的区别](https://blog.csdn.net/weixin_37766296/article/details/80459241)

参考2 : [白话Https](https://www.cnblogs.com/xinzhao/p/4949344.html)

[toTop](#jump)

# linux nginx配置https


参考1 : [linux nginx配置https](https://blog.csdn.net/w410589502/article/details/72833283)


[toTop](#jump)

# GET POST区别

![](/img/get_post.PNG)

[toTop](#jump)

# HTTP 状态码

## 1XX 信息

```
100 Continue ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。
```
## 2XX 成功

```
200 OK

204 No Content ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

206 Partial Content ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。
```

## 3XX 重定向

```
301 Moved Permanently ：永久性重定向

302 Found ：临时性重定向

303 See Other ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

304 Not Modified ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

307 Temporary Redirect ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。
```

## 4XX 客户端错误

```
400 Bad Request ：请求报文中存在语法错误。

401 Unauthorized ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。

403 Forbidden ：请求被拒绝。

404 Not Found
```

## 5XX 服务器错误

```
500 Internal Server Error ：服务器正在执行请求时发生错误。

503 Service Unavailable ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。
```
[toTop](#jump)
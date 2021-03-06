# 1.TCP与UDP的区别

TCP协议是一种可靠的、面向连接的协议:保证通信主机之间有可靠的字节流传输，完成流量控制功能，协调收发双方的发送与接收速度，达到正确传输的目的。

UDP是一种不可靠、无连接的协议:其特点是协议简单、额外开销小、效率较高，但是不能保证传输是否正确。

# **2.TCP对应的协议和UDP对应的协议**

**TCP**

1. HTTP协议：从Web服务器传输超文本到本地浏览器的传送协议，默认使用80端口。
2. FTP：文件传输协议，默认使用21端口。
3. SMTP：简单邮件传送协议，用于发送邮件，默认使用25号端口。
4. POP3：和SMTP对应，POP3用于接收邮件，默认使用110端口
5. Telnet：一种用于远程登陆的端口，用户可以以自己的身份远程连接到计算机上，通过这种端口可以提供一种基于DOS模式下的通信服务，默认使用23端口

**UDP**

1. DNS：域名解析服务，将域名地址转换为IP地址，默认使用53号端口。
2. TFTP(Trival File Transfer Protocal)：简单文件传输协议，默认使用69号端口。
3. SNMP：简单网络管理协议，默认使用161号端口，是用来管理网络设备的。



# 3.**TCP三次握手**

三次握手:TCP是通过报文段进行通信的，在建立TCP连接过程中，需要客户端和服务端总共发送3个报文段以确认连接的建立。

第一次握手：Client将标志位SYN置为1，随机产生一个序号（seq）J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。

第二次握手：Server接收到报文段，生成一个新的报文段发送给Client，新报文段中：SYN = 1，ACK = 1，序号（seq）= J+1，确认序号（ack） = k（k为随机生成）；Server进入SYN_RCVD状态

第三次握手：Client收到报文段，对报文段进行校验（ACK是否为1，ack是否为J+1），校验通过生成一个新的报文段发送给Server：ACK = 1，ack = K+1，此时Client进入ESTABLISHED状态；Server对接收到的报文段校验，校验通过进入ESTABLISHED状态。

#### **TCP为什么是三次握手不是两次**

TCP的三次握手最主要是防止已过期的连接再次传到被连	接的主机：如果使两次握手，有可能能第一次握手客户端发送的报文段过了很久才到达服务器端，此时客户端已经不需要连接了，如果服务器端建立了连接，就会造成服务器资源的浪费



# 4.TCP四次挥手

由于TCP连接时全双工的，因此，每个方向都必须要单独进行关闭，这一原则是当一方完成数据发送任务后，发送一个FIN来终止这一方向的连接，收到一个FIN只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个TCP连接上仍然能够发送数据，直到这一方向也发送了FIN。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭，上图描述的即是如此。

第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态

第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。

第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。

第四次挥手:Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

# 5.**为什么建立连接是三次握手，而关闭连接却是四次挥手呢？**

这是因为服务端在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

# 6.**HTTP协议包括哪些请求？**

常用的请求有：get，post，update，delete，head，options。GET：请求读取由URL所标志的数据 POST：给服务器添加或者更新数据 PUT：在给定的URL下存储一个文档 DELETE：删除给定的URL所标志的资源 OPTIONS：服务器针对特定资源所支持的HTTP请求方法 HEAD：向服务器索要与GET请求相一致的响应，只不过响应体将不会被返回。这一方法可以在不必传输整个响应内容的情况下，就可以获取包含在响应消息头中的元信息。

**HTTP中POST与GET的区别**

GET是将请求参数加到URL中，POST是将请求数据放在请求体中。

GET传送的数据量较小，不能超过2KB，POST传送的数据量较大，默认为不受限制。



# 7.**HTTP协议的格式**

#### **HTTP请求**

请求行：三个部分组成：第一部分是请求方法，第二部分是请求网址，第三部分是HTTP版本

请求头：请求头(request header) ；普通头(general header) ；实体头(entity header)

内容：通常来说，由于GET请求往往不包含内容实体，因此也不会有实体头。 第三部分内容只在POST请求中存在，因为GET请求并不包含任何实体

#### **HTTP响应**

状态行：第一部分是HTTP版本，第二部分是响应状态码，第三部分是状态码的描述

HTTP头：响应头(response header) ；普通头(general header) ；实体头(entity header)

内容：响应内容就是HTTP请求所请求的信息。这个信息可以是一个HTML，也可以是一个图片

#### **Http和Socket区别**

Http是一个协议，Socket是一个接口

Http可能是基于Socket的

Socket可以维持一个长连接，http是请求响应式，通常Socket效率高



# 8.**COOKIE和SESSION有什么区别**？

因为HTTP是无状态的，所以需要某种机制识别用户

Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；

Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。



# 9.**为什么 HTTPS 比 HTTP 安全**

HTTP 请求过程中，客户端与服务器之间没有任何身份确认的过程，数据全部明文传输，“裸奔”在互联网上，所以很容易遭到黑客的攻击。

而 HTTPS 实际上是带有 SSL 的 HTTP（HTTP + SSL=HTTPS）。当您在浏览器的地址栏中看到 HTTPS 时，这就意味着与该网站的所有通信都将被加密，整个访问过程更加安全。

HTTPS 的安全性往往体现在三个方面：

- 服务器身份验证，通过服务器身份验证，用户可以明确当前它正在与对应的服务器进行通信。
- 数据机密性，其他方无法理解发送的数据内容，因为提交的数据是加密的。
- 数据完整性，传输会携带 Message Authentication Code（MAC）用于验证，因此传输的数据不会被另一方更改。

HTTP请求会以明文的形式直接发送，既然是明文的形式，对于协议命令和语法有基本了解的人，只要监控了请求发送的过程，就能获取并读懂请求的意义。因此用 HTTP 的方式发送密码一类的数据时，安全性极低。

相对的，HTTPS 使用了 SSL（或 TLS）来加密 HTTP 请求和响应，因此在上面的示例中，监控请求的人将会看到一串随机的数字，而不是可读性的文本。其中加密过程采用的 SSL（安全套接字层）这一标准的安全技术，涵盖了非对称密钥和对称密钥。

 **HTTPS 优点：**

- 最大限度地提高 Web 上数据和事务的安全性；
- 加密用户敏感或者机密信息；
- 提高搜索引擎中的排名
- 避免在浏览器中出现“不安全”的提示；
- 提升用户对网站的信赖。

**缺点：**

- HTTPS 协议在握手阶段耗时相对较大，会影响页面整体加载速度；
- 在浏览器和服务器上会更多的 CPU 周期来加密/解密数据；
- SSL 证书一般都需要支付一定费用来获取，并且费用往往不低；
- 并不是绝对意义上的安全，在网站遭受攻击，服务器被劫持时，HTTPS 基本起不到任何安全防护作用。
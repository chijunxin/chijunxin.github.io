# Websocket协议解析



<!--more-->



# 概要
说到WS，不得不提HTTP,HTTP是基于TCP，面向文本的，无状态的，半双工通信协议。由于HTTP1.0的缺陷，后面版本做相应的优化：

* 每次请求都要建立TCP连接，并且请求一个文档需要两倍RTT(一个是TCP握手，一个是请求和接收文档)。针对这种情况HTTP1.1引入了keep-alive机制，实现了建立一次TCP连接，可发送多次请求。
* HTTP是明文传输，所以有一个中间人攻击的漏洞。针对这种情况引入了HTTPS(HTTP+TLS),实现了防窃听（数据加密）、防冒充（身份验证）、防篡改（完整性校验）。
* HTTP的请求一般是非流水线模式【收到上一个请求的响应后才能发下一个请求】（注意现代浏览器并不启用流水线模式，实现复杂，问题多），这里有队头阻塞问题（现代浏览器一般通过开多个tcp连接来进行一定程度上的规避），所以有了HTTP2。
* HTTP2只是解决了应用层的队头阻塞问题，但是TCP有丢包重传等机制，所以传输层也有队头阻塞问题，这个时候就要HTTP3登场了，引入 QUIC 协议，基于UDP,可以完美解决队头阻塞问题。

通过上面分析HTTP迭代的原因，那么WS的出现是为了解决HTTP什么问题呢？

`针对HTTP请求-响应这种半双工通信模式，既要实现全双工通信`

为什么要这样呢？主要是因为在没有WS之前，HTTP要实现IM,服务端事件推送等实时场景是很麻烦的，常见的有短轮询，长轮询，SSE（Server Send Event)，针对实时通讯的场景方案都不是很优秀，所以有了WS，它最初是在 HTML5 中引入的。经过多年发展后，该协议慢慢被多个浏览器支持，RFC 在 2011 年就把该协议作为一个国际标准，叫 rfc6455。

也许是为了修复HTTP缺陷而诞生的，所以WS的建立是依赖HTTP的，同样规定默认使用80,443端口，当然其错误码是与HTTP不重合的。


## WS原理
WebSocket 是一种基于TCP长连接，面向报文（二进制），支持全双工通信的网络协议。其与HTTP是平级的，相比HTTP1.1,优点如下：

* 支持全双工通信，实时性更强；
* 客户端与服务端通信的最小单位是帧（frame），由1个或多个帧组成一条完整的消息。即发送端将消息切割成多个帧，并发送给接收端；接收端接收消息帧，并将关联的帧重新组装成完整的消息；
* 长连接，有状态；
* 较少的控制开销。连接创建后，WS客户端、服务端进行数据交换时，协议控制的数据包头部较小，不像HTTP每次请求都要携带那么多头；
* 支持扩展。ws协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议（比如支持自定义压缩算法等）。

HTTP1.1的非流水线请求模式如下图：

![](/images/websocket协议解析/1.png)

可以看到HTTP1.1 强调请求与响应一对一，一个请求的响应回来了才会发出下一个请求。

WS请求模式如下图：

![](/images/websocket协议解析/2.png)

可以看到WS中的req与rsp的发起与返回非常自由，没有一对一的概念。

### 帧格式
WebSocket主要是解决HTTP无法实时通信的问题，所以没有HTTP2 的多路复用，优先级等复杂功能，所以二进制帧格式简单：

![](/images/websocket协议解析/3.png)

可以看到帧头是16bit，2字节：

* 第一个字节：
前四位是标志位，第一个标志位是FIN,表示报文结束，因为一个消息可能被拆分成多帧，当接收方收到携带FIN标志位的帧时，就会把前面的帧拼起来组成完整的消息。后三位是保留标志位，必须设置为0，除非想要自定义扩展。
后四位是opcode,也就是帧类型，比如0表示连续帧，1表示帧内容是文本，2表示帧内容是二进制，8表示关闭连接，9和10表示ping和pong等
* 第二个字节：
它的第一位是掩码标志位 MASK，表示帧内容是否使用异或操作（xor）做简单的加密，当该位被设置为 1 时表示加密，那么后续会设置4字节的Masking-key，设置为 0 时表示不加密，就不会携带Masking-key了。如果加密了，那么必须解密才能得到正确内容。目前的 WebSocket 标准规定，客户端发送数据必须使用掩码加密，而服务器发送则不使用掩码加密。
后面7位表示Payload Length，也就是有效负载或者说有效业务消息的长度，并且采用的是大端存储。但是7位大表示127，如果payload data大于127字节咋办？？
所以就引入了 Extended payload length 来表示真是的Payload Length：

Payload Length ==127，则 后面8字节表示 Extended payload length；

![](/images/websocket协议解析/4.png)

Payload Length ==126，则 后面2字节表示 Extended payload length；

![](/images/websocket协议解析/5.png)

Payload Length < 126，则 没有 Extended payload length；

![](/images/websocket协议解析/6.png)

可知最大payload data长度 可以用8字节表示。


## WS实战
* WS是基于HTTP协议实现的。
* WSS是基于HTTPS协议实现的,即完成tls握手后再进行协议升级。

WS建立复用了 HTTP 的握手请求过程。
客户端通过 HTTP 请求与 WebSocket 服务端协商升级协议。协议完成后，后续的数据交互则遵循 WebSocket 的协议。

### 客户端发起协议升级请求
```bash {open=true, lineNos=false, wrap=false}
GET /websocket HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Sec-WebSocket-Version: 13 #表示WS的版本。如果服务端不支持该版本，需要返回一个Sec-WebSocket-Versionheader头，里面包含服务端支持的版本号
Origin: http://www.jsons.cn
Sec-WebSocket-Extensions: permessage-deflate
Sec-WebSocket-Key: 9zQPEx8m0sqMkV1vanwJIA== #与服务端响应首部的Sec-WebSocket-Accept是配套的，提供基本的防护，比如恶意的连接，或者无意的连接(比如特殊的HTTP请求被意外识别成WS)
Connection: keep-alive, Upgrade   #告诉服务端要升级协议了
Sec-Fetch-Dest: websocket
Sec-Fetch-Mode: websocket
Sec-Fetch-Site: cross-site
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket             #告诉服务端要升级成WS协议

```

### 服务端响应协议升级
```bash {open=true, lineNos=false, wrap=false}
HTTP/1.1 101 Switching Protocols  #状态码 101 表示协议切换成功
Upgrade: websocket  #表示升级到WS协议
Connection: Upgrade  #表示协议升级
Sec-WebSocket-Accept: MzzLu4vmstovah28VVOvEgveq8o=  #根据客户端请求首部的 Sec-WebSocket-Key 计算出来
#将 Sec-WebSocket-Key 跟 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 拼接。
#通过 SHA1 计算出摘要，并转成 base64 字符串。计算公式如下：
# Base64(sha1(Sec-WebSocket-Key + 258EAFA5-E914-47DA-95CA-C5AB0DC85B11))

```
协议升级完成后就完全遵循WS协议发送接收数据了。

### 核心事件
WebSocket协议也是事件驱动的，客户应用程序不需要轮序服务来得到更新的数据，消息和事件将在服务器发送它们的时候异步到达。

1. open 连接建立时触发
2. message 接收/发送消息时触发
3. error 通信发生错误时触发
4. close 连接关闭时触发

### 心跳保活
这个其实在1.1小节已经说过了，就是opcode类型，通过ping和pong来实现，不需要自己再写保活接口了，只需要使用ping/pong即可。

以go [gorilla/websocket](https://github.com/gorilla/websocket)为例：
``` go { open=true }
func WebSocket(c *gin.Context) {
   //支持协议升级
	conn, err := (&websocket.Upgrader{CheckOrigin: func(r *http.Request) bool { return true }}).Upgrade(c.Writer,      	c.Request, nil)
	if err != nil {
		http.NotFound(c.Writer, c.Request)
		return
	}
	//添加保活机制
	conn.SetPongHandler(func(appData string) error {
		library.ZapDebug(appData)
		//check something
		return nil
	})
	//deal websocket connect
	for {
	  //接收消息
		mt, msg, err := conn.ReadMessage()
		if err != nil { //错误，打Trace，因为可能是主动或者网络问题
			library.ZapDebug("ReadMessage err:%v", err)
			continue
		}
		//这里可以go func(){}() 包一下解决并发问题
		{
			//业务处理
			library.ZapDebug("mt=%v,msg=%v", mt, string(msg))
			//响应消息
			err = conn.WriteMessage(mt, []byte(fmt.Sprintf("hello:%s", msg)))
			if err != nil {
				library.ZapDebug("WriteMessage err:%v", err)
			}
		}
	}
}

```

### 注意WS无多路复用
HTTP2为了提高传送效率，引入了多路复用，即流的概念，每个请求都有自己专属的流id，来保证并发请求下服务端响应的内容精准匹配到对应的请求。从在1.1节中看到WS显然不支持这样的情况，即WS下先后发出a,b两个请求，WS并不保证a的返回一定先于b。这一点在使用中需要注意的。

还是以下面的代码为例：
``` go { open=true}
func WebSocket(c *gin.Context) {
	conn, err := (&websocket.Upgrader{CheckOrigin: func(r *http.Request) bool { return true }}).Upgrade(c.Writer, c.Request, nil)
	if err != nil {
		http.NotFound(c.Writer, c.Request)
		return
	}
	//保活
	conn.SetPongHandler(func(appData string) error {
		library.ZapDebug(appData)
		//check something
		return nil
	})
	// websocket connect
	for {
		mt, msg, err := conn.ReadMessage()
		if err != nil { //客户端错误，打Trace，因为可能是主动或者网络问题
			library.ZapDebug("err:%v", err)
			continue
		}
		//这里可以go func(){}() 包一下解决并发问题
		go func() {
			//业务处理
			library.ZapDebug("mt=%v,msg=%v", mt, string(msg))
			t, _ := strconv.ParseInt(string(msg), 10, 64)
			time.Sleep(time.Duration(t) * time.Second) //sleep一下，人为实现后面的请求比前面的请求先返回
			//响应消息
			err = conn.WriteMessage(mt, []byte(fmt.Sprintf("hello:%s", msg)))
			if err != nil {
				library.ZapDebug("err:%v", err)
			}
		}()
	}
}

```

[测试网址](http://www.jsons.cn/websocket/)

![](/images/websocket协议解析/7.png)

可以观察到 WS下先后发出a,b两个请求，会出现 b请求先于a请求先返回的。

原因：

TCP是面向字节流的，WS协议简单，可以说只是在TCP之上提供了数据拆包组包的功能，`那么WS在收包时只能保证消息是完整的，并不管是谁发出的，只要有完整消息就会触发message 事件接收消息`。

## 总结
经过上述内容，可以知道WS可以看做是对HTTP打的补丁，主要是为了解决HTTP半双工通信弊端，侧重处理数据实时通信场景。

这里再说下WS为什么没有替代HTTP，反而后面出了一样支持长连接的HTTP2 ？

1. WS协议简单，可以说只是在TCP之上提供了数据拆包组包的功能。所以对于静态资源的请求效率是低下的，而不像HTTP2提供了多路复用，请求优先级等特性来提高传输效率，增强页面渲染。
2. 长连接，全双工通信是WS相比于HTTP的一大优势，其实HTTP2也有，但奈何浏览器为了提高页面渲染效率往往还是同时开多个TCP连接。可以这样说，WS与HTTP各有优势，在页面渲染和短请求场景下选HTTP，在实时通信，如IM等场景下选WS,二者结合使用最好。
3. 长连接也并不是银弹，随着在线用户增加，相同资源下，其最大在线用户数量理论上不如HTTP。
4. 长连接的负载均衡不如短连接灵活，简单。
5. WS并不天然支持请求-响应一对一模式，但是网页渲染，接口请求这样的场景是需要，要想实现就要人为控制，所以此类场景下不如HTTP香。

可以这样说，如果不是浏览器是个沙盒环境，不允许用户使用TCP，说不定WS压根不会诞生。




<br><br><br>原文链接：<https://blog.csdn.net/weixin_38597669/article/details/131885507>


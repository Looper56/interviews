# 服务器相关

### 输入url到页面展示
#### 首先解析本地host->应用层DNS服务器解析->应用层HTTP请求报文->传输层建立TCP链接->网络层使用IP选择络线->物理层传输数据->服务器处理反向代理->服务器返回HTTP响应->浏览器渲染

### TCP连接三次握手 （syn：Synchronize Sequence Numbers）（传输层，数据传输控制协议）
#### 第一次：建立连接，客户端发送syn包，待服务器确认
#### 第二次：服务器收到syn包，确认客户syn
#### 第三次：客户端收到服务器syn包，此时向服务器发送确认包ACK，客服端服务端进入ESRABLEIDHSHED状态
#### 即对每次发送数据量怎么跟踪进行协商数据段的发送和接受同步的，根据所接收的数据量而确定数据确认数，接收完毕后何时撤销联系，并建立虚链接

### TCP断开四次握手
#### 第一次：TCP发送FIn，用来关闭服务端连接，客服端进程发送连接释放报文
#### 第二次：服务端收到这个FIN，发回一个ACK，服务端连续发送释放报文
#### 第三次：服务端发送FIN到客服端，服务端关闭客服端连接
#### 第四次：客服端发送ACK报文确认，确认序号+1，完成关闭

### Http和Https区别
#### http:超文本传输协议，一种用于分布式，协作式和超媒体系统的应用层协议，协议以明文方式发送，不提供加密，默认工作在TCP协议80端口
#### https:超文本传输安全协议，透过计算机网络进行安全通信的传输协议，利用SSL(安全加密隧道协商)/TLS加密数据包，默认工作在TCP协议443端口

### TCP和UDP
#### TCP是面向连接的，UDP是无连接的
##### TCP需要先建立连接，然后进行数据交互：服务器的listen()/accept()函数确保连接
##### Linux内核协议栈为TCP连接创建两个队列
#### TCP是可靠的，UDP是不可靠的
##### TCP有数据校验，确认序列号，丢弃重复数据包，确认应答机制。超时重传，流量控制，拥塞控制
##### UDP如果校验出错，将会丢弃该报文
#### TCP是面向字节流的，UDP是面向数据报文的
##### TCP以字节为发送单位，数据包拆分为多个数据包
##### TCP先将数据放在发送缓存区，等待达到一定大小发送，也可以直接发送
##### UDP发送需要固定长度，或者将其裁剪到合适长度
#### TCP只支持点对点通信，UDP支持一对一，一对多，多对多
##### TCP需要双方建立连接，需要点对点通信
#### TCP报文首部20个字节，UDP首部8个字节
#### TCP拥有塞控机制，UDP没有
##### 拥塞机制降低重传和丢包率：满启动，拥塞避免，快重传+快恢复
#### TCP协议下双方接收发送缓冲区都有，UDP并无实际意义发送缓存区，但有接收缓冲区
#### 如何选择TCP或UDP：对实时性要求比较高的情况选择UDP(游戏/媒体通信/实时视频流)，其余部分可选择TCP

### TCP对于应用层协议
#### FTP：定义文件传输，使用21端口
#### Telnet：远程登录端口，基于DOS模式下通信服务，使用23端口
#### SMTP：邮件发送协议，使用25端口
#### POP3:与SMTP对应，接收邮件，使用110端口

### UDP对应应用层协议
#### DNS：用于域名解析，将域名地址转换为IP地址，使用53端口
#### SNMP：网络管理协议，设备无连接服务体现优势，使用161端口
#### TFTP：文件传输协议，UDP服务，使用69端口

### RPC和GRPC
#### RPC(Remote Procedure Call)是一种通过网络从远程计算机程序请求服务，不需要了解底层网络细节程序通信协议，RPC协议构建于TCP和UDP，或者HTTP上(Go net/rpc包)
#### GRPC 高性能，开源，通用的RPC框架，基于HTTP2协议，默认采用Protocol Buffers数据序列化协议，消息结构

### HTTP动词
#### GET(SELECT):从服务器取出资源，一项或多项
#### POST(CREATE):在服务器创建资源
#### PUT(UPDATE):在服务器更新资源，客户端提供改变后的完整资源
#### PATCH(UPDATE):在服务器更新资源，客户端提供改变的属性
#### DELETE(DELETE):从服务器删除资源
#### HEAD：获取资源元数据
#### OPTIONS：获取信息，资源哪些属性是可以修改的

### 对称加密与非对称加密
#### 对称密钥加密是指加密和解密用同一个密钥公式，发送密钥问题，不安全
#### 非对称加密使用公钥和私钥，公钥随意发布，速度慢

### DDos攻击：
#### 客服端向服务发送请求链接数据包
#### 服务端向客服端发送确认数据包
#### 客户端不向服务端发送确认数据包，服务器一直等待

### DDos预防：
#### 限制同时打开SYN半链接数目
#### 缩短SYN链接Time out时间
#### 关闭不必要服务区

### HTTP协议无状态
#### 优点：解放服务器，每一次链接点到为止，避免链接占用
#### 缺点：每次请求会传输大量重复内容信息

### CDN是什么？
#### 内容发布网络，将网站内容通过中心平台发布到部署在各地的边缘服务器，再通过负载均衡技术将用户的请求转发到就近的服务器上获取所需内容，降低网络堵塞，提供刚问网络响应速度和命中率
#### 过程：首先本地dns解析，请求cname指向cdn专用的dns服务器->dns返回全局负载均衡服务器->用户请求到服务器根据ip返回负载均衡服务器->服务器就近选择，如果没有会去上一级缓存服务器，就近原则
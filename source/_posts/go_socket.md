---
title: Golang网络编程（一）
tags:
 - Golang
 - socket
---
## 1. 前言

​	最近工作当中用Python写了非常多的socket代码，用于和底层的设备之间进行交互。然而我的方式比较原始，自己在一个基础的socket上不断地进行扩展。总所周知，Python的网络编程界有一个大名鼎鼎的[Twisted](https://www.twistedmatrix.com/trac/)框架，Twisted是已经一个维护了十余年的成熟项目，基于事件驱动设计的高性能网络编程框架。奈何这个框架的学习成本比较高，再由于笔者最近在学习Go语言，所以想着不如在Go语言中折腾一下网络编程，以下就是笔者学习阶段的一些总结。
<!-- more -->

## 2. Socket编程起源

socket起源于Unix，而Unix/[Linux](http://lib.csdn.net/base/linux) 基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式 来操作。Socket就是该模式的一个实现，socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）

在 Unix/Linux 中的 Socket 编程主要通过调用 `listen`, `accept`, `write` `read` 等函数来实现的. 具体如下图所示:

![EXEMo8.png](https://s2.ax1x.com/2019/05/18/EXEMo8.png)

## 3. 再回首——Python中的socket

在进入到Go语言的世界前，和我再来回顾一下，Python中Socket编程的基础实例：

Server.py

``````python
# Echo server program
import socket

HOST = ''                 # Symbolic name meaning all available interfaces
PORT = 50007              # Arbitrary non-privileged port

sock_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock_server.bind((HOST, PORT))

sock_server.listen(1) #开始监听，1代表在允许有一个连接排队，更多的新连接连进来时就会被拒绝
conn, addr = sock_server.accept() #阻塞直到有连接为止，有了一个新连接进来后，就会为这个请求生成一个连接对象

with conn:
    print('Connected by', addr)
    while True:
        data = conn.recv(1024) #接收1024个字节
        if not data: break #收不到数据，就break
        conn.sendall(data) #把收到的数据再全部返回给客户端
``````

Client.py

``````python
# Echo client program
import socket

HOST = 'localhost'    # The remote host
PORT = 50007              # The same port as used by the server

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((HOST, PORT))
client.sendall(b'Hello, world')

data = client.recv(1024)

print('Received',data)
``````

显而易见，Python的代码哲学还是有优点的。好了，整理好心情，和Python道声再见，我们进入Go的世界。

## 4. Go SOCKET基础概念

### 4.1 IP类型

net包中定义的IP类型直接就是byte数组：

``````go
type IP []byte
``````

我们可以使用`func parseIP(s string) IP`来把一个IP地址转换成IP类型：

``````go
ipAddr := "192.168.1.79"
addr := net.ParseIP(ipAddr)
if addr == nil{
	fmt.Println("unavaliable addr")
}else{
	fmt.Println(addr.To16())
}
``````

### 4.2 函数

#### 4.2.1 **funcResolveTCPAddr(net, addr string) (\*TCPAddr, error)**

ResolveTCPAddr`函数的功能是解析TCP连接的地址，包含`ip`和`port

- `net`：`tcp` `tcp4` `tcp6`三选一，分别表示TCPv4，TCPv6和任意，默认是`tcp4`
- `addr`：主机的地址，可以是`[ip+port]`，也可以是`[domain+port]`，可以省略主机部分，表示本机地址

返回一个`*TCPAddr`类型 ，表示一个TCP连接地址：

``````go
type TCPAddr struct {
	IP   IP
	Port int
	Zone string // IPv6 scoped addressing zone
}
``````

#### 4.2.2 func ResolveIPAddr(net, addr string) (\*IPAddr, error)

`ResolveIPAddr`函数的功能是解析ip地址：

- `net`：`ip`  `ip4` `ip6` 分别代表IPv4，IPv6以及任意。默认留空表示`ip4`
- `addr` ：IP地址

返回一个`*IPAddr`结构：

``````go
type IPAddr struct {
	IP   IP
	Zone string // IPv6 scoped addressing zone
}
``````

#### **4.2.3 func Dial(network, address string) (Conn, error)**

`Dial`函数的功能是建立一个连接：

- `network`： 如果是TCP连接，对应`tcp` `tcp4` `tcp6；如果是IP连接，对应ip ip4 ip6`。对于ip连接，需要在后面加一个冒号然后注明协议号或者协议名字
- `address`：连接的地址，`ip+port`或`domain+port` 形式，也可以省略主机地址表示本地地址`217.0.0.1`

返回一个`net.Conn`接口对象，包含了连接的信息，我们可以使用该对象的`Write()`和`Read()`对连接进行读写。

``````go
type Conn interface {
	Read(b []byte) (n int, err error)
	Write(b []byte) (n int, err error)
	Close() error
	LocalAddr() Addr
	RemoteAddr() Addr
	SetDeadline(t time.Time) error
	SetReadDeadline(t time.Time) error
	SetWriteDeadline(t time.Time) error
}
``````

与这个函数相对应的两个函数：

- `func DialTCP(net string, laddr, raddr *TCPAddr) (*TCPConn, error)`
- `func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)`

分别表示建立TCP请求和IP请求，中间多的`laddr`表示本地的地址，一般为`nil` 。

#### **4.2.4 func (c \*conn) Write(b []byte) (int, error)**

向`conn`连接对象中写入数据，即发送数据给对方，写入的数据是`[]byte`类型，成功将返回发送的数据包字节数。

``````go
n, err := tcpCoon.Write([]byte("HelloWorld"))
if err != nil{
	fmt.Println(err)
	return
}
``````

#### **4.2.5 func (c \*conn) Read(b []byte) (int, error)**

从conn连接对象中读取数据，成功将返回读取到的字节数。

``````go
recvData := make([]byte, 2048)
n, err = tcpCoon.Read(recvData)
if err != nil{
   fmt.Println(err)
   return
}
fmt.Println(string(recvData))
``````

#### **4.2.6 func Listen(net, laddr string) (Listener, error)**

Listen函数在服务端使用，让服务端开始监听。

- `net` ：和上面一样，可以是`tcp` `ip`相关的值
- `laddr` ：要监听的地址，`ip+port`省略主机地址将使用本机地址`127.0.0.1`

相应的两个函数：

- `func ListenTCP(net string, laddr *TCPAddr) (*TCPListener, error)`：监听TCP连接
- `func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)`：监听IP连接

#### **4.2.7 func (l \*TCPListener) Accept() (Conn, error)**

服务端开始监听需要使用`Accept`函数来接受客户端连接，此时服务端将进入阻塞状态。

相应的还有一个

- `func (l *TCPListener) AcceptTCP() (*TCPConn, error)`

## 5. Go 基础scoket代码实例

经过上面的介绍，相信大家对Go的socket编程已经有了一些了解。现在我们就来写一个server和client，实现功能：client发送数据到server，server将数据转成大写后返回。

server.go

``````go
package main
 
import (
	"net"
	"fmt"
	"strings"
)
 
func main(){
	tcpAddr, err := net.ResolveTCPAddr("tcp4", "localhost:8080") //创建一个TCPAddr
	if err != nil{
		fmt.Println(err)
		return
	}
 
	tcpLinstener, err := net.ListenTCP("tcp4", tcpAddr) //开始监听
	if err != nil{
		fmt.Println(err)
		return
	}
	fmt.Printf("Start listen:[%s]
", tcpAddr)
 
	tcpCoon, err := tcpLinstener.AcceptTCP()  //阻塞，等待客户端连接
	if err != nil{
		fmt.Println(err)
		return
	}
	defer tcpCoon.Close()  //记得关闭连接对象
 
	data := make([]byte, 2048)
	n, err := tcpCoon.Read(data)  //客户端连接后，开始读取数据
	if err != nil{
		fmt.Println(err)
		return
	}
 
	recvStr := string(data[:n])
	fmt.Println("Recv:", recvStr)
	tcpCoon.Write([]byte(strings.ToUpper(recvStr)))  //转换成大写后返回客户端
}
``````

client.go

``````go
package main
 
import (
	"net"
	"fmt"
)
 
func main(){
	tcpAddr, err := net.ResolveTCPAddr("tcp4", "localhost:8080")  //TCP连接地址
	if err != nil{
		fmt.Println(err)
		return
	}
 
	tcpCoon, err := net.DialTCP("tcp4", nil, tcpAddr)  //建立连接
	if err != nil{
		fmt.Println(err)
		return
	}
	defer tcpCoon.Close()  //关闭
 
	sendData := "helloworld"
	n, err := tcpCoon.Write([]byte(sendData))  //发送数据
	if err != nil{
		fmt.Println(err)
		return
	}
	fmt.Printf("Send %d byte data success: %s
", n, sendData)
 
	recvData := make([]byte, 2048)
	n, err = tcpCoon.Read(recvData)  //读取数据
	if err != nil{
		fmt.Println(err)
		return
	}
	recvStr := string(recvData[:n])
	fmt.Printf("Response data: %s", recvStr)
``````

## 6. 小结

至此，相信大家对Go的网络编程已经有了一个大致的了解。

后续的计划是

- Go并发socket编程
- Go的socket编程粘包处理
- 挑选一个成熟的Go网络编程框架进行学习

## 参考资料

> 马谦的博客：<https://www.dyxmq.cn/code/golang/golang-socket-2.html>
>
> 始于珞尘：<https://juejin.im/entry/5aa8ebe46fb9a028de4467bd>
>
> 《Go语言程序设计》
---
title: Golang的网络编程（三）：解决粘包问题
tags:
 - Golang
categories:
 - 计算机技术
---

## **一、概述**

前面已经完成了一个完美的多并发CS模型，但美中不足的是没有解决粘包问题。
<!-- more -->

### **1.1 什么是粘包问题？**

在网络传输中，数据都是通过数据流来传输的，也就是以比特来传输。传输的过程中我们可能会遇到各种各样的问题导致数据传输异常，最常见的就是网络发送时延。网络时延会导致服务端此时收到的数据的时间有偏差，然后就导致数据接收数据的时间不一致。

可以看一个例子，修改上篇的服务端和客户端为以下内容：

``````go
for {
		data := make([]byte, 2048)
		n, err := conn.Read(data)
		if n == 0{
			fmt.Printf("%s has disconnect", conn.RemoteAddr())
			break
		}
		if err != nil{
			fmt.Println(err)
			continue
		}
		//fmt.Printf("Receive data [%s] from [%s]", string(data[:n]), conn.RemoteAddr())
		//修改上句为下面的
		fmt.Printf("Receive %d byte data : %s", n, string(data[:n]))
		//程序睡眠1ns，模拟网络时延
		time.Sleep(time.Nanosecond)
	}
``````

客户端改为以下：

``````go
func main(){
	conn, err := net.Dial("tcp", ":8899")
	if err != nil{
		fmt.Println(err)
		return
	}
	for i := 0; i < 100; i++{
		data := fmt.Sprintf("{"index":%d, "name":"maqian", "age":21, "company":"intely"}", i + 1)
		n, err := conn.Write([]byte(data))
		if err != nil{
			fmt.Println(err)
			continue
		}
		fmt.Printf("Send %d byte data : %s
", n, data)
	}
}
``````

以上我们发送了100条json到服务端，按照预想服务端将会输出100行json，但是实际上并不是：

![VAnK4f.png](https://s2.ax1x.com/2019/05/25/VAnK4f.png)

这个现象产生的原因是因为服务端每次读取数据之后将会休眠1ns，但是对于客户端来说，这1ns它还在一直传输数据，1ns的时间可能 发送了1条，也可能是2条，这个数量我们不知道是多少，也无法控制。于是就导致数据堆积，服务端再读取就会出问题了。与此同时，由于缓冲区有限，一次最多读取2048个字节，堆积的字节超过2048的也无法读取，只能留到下次读取，这种现象就是粘包问题。

## 二、解决办法

上面抛出了粘包的问题后，现在就要开始想办法处理了，怎么处理呢？这里就需要用到协议了，协议就是双方约定好的数据包格式， 让服务端知道从哪里开始读，读到哪里结束，这样就不会出错了。实现这个协议最简单的办法就是加上一个协议头和一个数据包长度 。

假设现在要发送`[0x11, 0x22, 0x33]`，约定协议头为`[0xaa, 0xbb]`，由于发送数据的长度是三个字节，所以经过客户端封装之后的数据就变成了`[0xaa, 0xbb, 0x03, 0x11, 0x22, 0x33]`

服务端收到数据后，先找`[0xaa, 0xbb]`的位置，然后根据他们的位置得到数据长度为`3`，于是再往后读三个字节就是真正的的数据 部分了。

## 三、实现

指定好了协议之后就可以开始实现了，为了方便，直接把这里写成一个对象：

``````go
type SocketUtil struct {	
    Coon		net.Conn
}
``````

包头的定义：

``````go
type PkgHeader struct {
	HeaderFlag	[2]byte
	DataLength	uint32
}
``````



包头包括协议头和数据长度，共六个字节。

### 3.1 数据发送时的封装

``````go
func (fd *SocketUtil) WritePkg(data []byte)(int, error){
	if fd == nil {
		return -1, errors.New(SOCKET_ERROR_SERVER_CLOSED)
	}
	if len(data) == 0{
		return 0, nil
	}
	buff := bytes.NewBuffer([]byte{})
	binary.Write(buff, binary.BigEndian, []byte{0xaa, 0xbb}) //添加协议头
	binary.Write(buff, binary.BigEndian, uint32(len(data))) //添加长度
	binary.Write(buff, binary.BigEndian, data) //数据部分
	allBytes := buff.Bytes()
	return fd.writeNByte(allBytes)
}
``````

`writeByte()`的实现

`````go
func (fd *SocketUtil) writeNByte(data []byte)(int, error){
	n, err := fd.Coon.Write(data)
	if err != nil{
		return -1, err
	}else{
		return n, nil
	}
}
`````

### 3.2 接收数据时解包

``````go
func (fd *SocketUtil) ReadPkg()([]byte, error){
	if fd == nil || fd.Coon == nil{
		return nil, errors.New(SOCKET_ERROR_SERVER_CLOSED)
	}
	head, err := fd.readHead() //先读取数据头
	if err != nil{
		return nil, err
	}
	//数据头和约定不一样，报错
	if head.HeaderFlag != [2]byte{0xaa, 0xbb}{
		return nil, errors.New("Head package error")
	}
	//读取指定长度的数据
	datas, err := fd.readNByte(head.DataLength)
	if err != nil{
		return nil, err
	}
	return datas, nil
}
``````

`readHead()`的实现：

``````go
func (fd *SocketUtil) readHead()(*PkgHeader, error){
	data, err := fd.readNByte(HeaderLength)
	if err != nil{
		return nil, err
	}
	h := PkgHeader{}
	buff := bytes.NewBuffer(data)
	binary.Read(buff, binary.BigEndian, &h.HeaderFlag) //读取0xaa 0xbb连个字节
	binary.Read(buff, binary.BigEndian, &h.DataLength) //读取四个字节的长度
	return &h, nil
}
``````

`readNByte()`的实现：

``````go
func (fd * SocketUtil) readNByte(n uint32)([]byte, error){
	data := make([]byte, n)
	for x := 0; x < int(n) ;{
		length, err := fd.Coon.Read(data[x:]) //有数据则读，没有则阻塞
		if length == 0{
			return nil, errors.New(SOCKET_ERROR_CLIENT_CLOSED)
		}
		if err != nil{
			return nil, err
		}
		x += length
	}
	return data, nil
}
``````

### 3.3 完整代码

``````go
package common
 
import (
	"net"
	"errors"
	"bytes"
	"encoding/binary"
)
 
 
 
type PkgHeader struct {
	HeaderFlag	[2]byte
	DataLength	uint32
}
 
const(
	HeaderLength = 6
)
 
const(
	SOCKET_ERROR_CLIENT_CLOSED  = "Client has been closed"
	SOCKET_ERROR_SERVER_CLOSED  = "Server has been closed"
	SOCKET_ERROR_TIMEOUT		= "Timeout"
)
 
type SocketUtil struct {
	Conn		net.Conn
}
 
func (fd *SocketUtil) Init(conn net.Conn){
	fd.Conn = conn
}
 
func (fd *SocketUtil) WritePkg(data []byte)(int, error){
	if fd == nil {
		return -1, errors.New(SOCKET_ERROR_SERVER_CLOSED)
	}
	if len(data) == 0{
		return 0, nil
	}
	buff := bytes.NewBuffer([]byte{})
	binary.Write(buff, binary.BigEndian, []byte{0xaa, 0xbb})
	binary.Write(buff, binary.BigEndian, uint32(len(data)))
	binary.Write(buff, binary.BigEndian, data)
 
	allBytes := buff.Bytes()
 
	return fd.writeNByte(allBytes)
}
 
func (fd *SocketUtil) ReadPkg()([]byte, error){
	if fd == nil || fd.Conn == nil{
		return nil, errors.New(SOCKET_ERROR_SERVER_CLOSED)
	}
	head, err := fd.readHead()
	if err != nil{
		return nil, err
	}
	if head.HeaderFlag != [2]byte{0xaa, 0xbb}{
		return nil, errors.New("Head package error")
	}
	datas, err := fd.readNByte(head.DataLength)
	if err != nil{
		return nil, err
	}
	return datas, nil
}
 
func (fd *SocketUtil) readHead()(*PkgHeader, error){
	data, err := fd.readNByte(HeaderLength)
	if err != nil{
		return nil, err
	}
	h := PkgHeader{}
	buff := bytes.NewBuffer(data)
	binary.Read(buff, binary.BigEndian, &h.HeaderFlag)
	binary.Read(buff, binary.BigEndian, &h.DataLength)
	return &h, nil
}
 
func (fd * SocketUtil) readNByte(n uint32)([]byte, error){
	data := make([]byte, n)
	for x := 0; x < int(n) ;{
		length, err := fd.Conn.Read(data[x:])
		if length == 0{
			return nil, errors.New(SOCKET_ERROR_CLIENT_CLOSED)
		}
		if err != nil{
			return nil, err
		}
		x += length
	}
	return data, nil
}
 
 
func (fd *SocketUtil) writeNByte(data []byte)(int, error){
	n, err := fd.Conn.Write(data)
	if err != nil{
		return -1, err
	}else{
		return n, nil
	}
}
 
func (fd *SocketUtil) Close(){
	fd.Conn.Close()
}
``````

## 四、服务端

``````go
package main
 
import (
	"net"
	"fmt"
	"网络编程/并发/common"
)
 
func handle(conn net.Conn){
	defer conn.Close()
	fmt.Println("Connect :", conn.RemoteAddr())
 
	fd := common.SocketUtil{conn}
	for {
		data, err := fd.ReadPkg() //读取数据
		if err != nil{
			fmt.Println(err)
			break
		}
		fmt.Println(string(data))
	}
 
}
 
func main(){
	listener, err := net.Listen("tcp", ":8899")
	if err != nil{
		fmt.Println(err)
		return
	}
	fmt.Println("Start listen localhost:8899")
 
	for {
		conn, err := listener.Accept()
		if err != nil{
			fmt.Println(err)
			return
		}
		go handle(conn)
	}
}
``````

## 五、客户端

``````go
package main
 
import (
   "net"
   "fmt"
   "网络编程/并发/common"
)
 
func main(){
   conn, err := net.Dial("tcp", ":8899")
   if err != nil{
      fmt.Println(err)
      return
   }
   clntFd := common.SocketUtil{conn}
   for i := 0; i &lt; 100; i++{
      data := fmt.Sprintf("{"index":%d, "name":"maqian", "age":21, "company":"intely"}", i + 1)
      n, err := clntFd.WritePkg([]byte(data))
      if err != nil{
         fmt.Println(err)
         return
      }
      fmt.Printf("Send %d byte data : %s
", n, data)
   }
}
``````



## 六、运行

运行服务端再运行客户端就会发现，已经不和之前的一样了，整整齐齐，perfect！

![VAulIx.png](https://s2.ax1x.com/2019/05/25/VAulIx.png)

 
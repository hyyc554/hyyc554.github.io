---
title: Golang网络编程（二）：并发Server-Client
tags:
 - Golang
categories:
 - 计算机技术
  
date: 2020-02-04 18:35
---


## 一、概述

上一篇实现了一个server和client通信，完成了小写转大写的功能，但是是一个单任务式的响应：客户端发送连接接收响应，程序结束；服务端则接收数据响应数据也结束！就实际需要而言，并没有很大的用处，所以现在我们就给客户端和服务端添加上并发功能。
<!-- more -->

逻辑其实很简单，就是利用golang的gorutine，一旦来新的连接，就开启一个gorutine去处理，然后响应，直到客户端关闭连接。

## 二、服务端

``````go
package main
 
import (
	"net"
	"fmt"
	"strings"
)
 
func handle(conn net.Conn){
	defer conn.Close()  //关闭连接
	fmt.Println("Connect :", conn.RemoteAddr())
 
	for {
		//只要客户端没有断开连接，一直保持连接，读取数据
		data := make([]byte, 2048)
		n, err := conn.Read(data)
		//数据长度为0表示客户端连接已经断开
		if n == 0{
			fmt.Printf("%s has disconnect
", conn.RemoteAddr())
			break
		}
		if err != nil{
			fmt.Println(err)
			continue
		}
		fmt.Printf("Receive data [%s] from [%s]
", string(data[:n]), conn.RemoteAddr())
                //转大写
		rspData := strings.ToUpper(string(data[:n]))
		_, err = conn.Write([]byte(rspData))
		if err != nil{
			fmt.Println(err)
			continue
		}
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
		//开始循环接收客户端连接
		conn, err := listener.Accept()
		if err != nil{
			fmt.Println(err)
			return
		}
		//一旦收到客户端连接，开启一个新的gorutine去处理这个连接
		go handle(conn)  
	}
}
``````

## 三、客户端

``````go
import (
	"net"
	"fmt"
)
 
func main(){
	conn, err := net.Dial("tcp", ":8899")  //连接服务端
	if err != nil{
		fmt.Println(err)
		return
	}
	fmt.Println("Connect to localhost:8899 success")
	defer conn.Close()
 
	for{
		//一直循环读入用户数据，发送到服务端处理
		fmt.Print("Please input send data :")
		var a string
		fmt.Scan(&a)
		if a == "exit"{break}  //添加一个退出机制，用户输入exit，退出
 
		_, err := conn.Write([]byte(a)) 
		if err != nil{
			fmt.Println(err)
			return
		}
 
		data := make([]byte, 2048)
		n, err := conn.Read(data)
		if err != nil{
			fmt.Println(err)
			continue
		}
		fmt.Println("Response data :", string(data[:n]))
	}
 
}
``````



## 四、运行
此时，我们开启一个服务端，开启两个客户端进行测试：
服务端：
![VCpkcQ.png](https://s2.ax1x.com/2019/05/22/VCpkcQ.png)
客户端：
![VCpVns.png](https://s2.ax1x.com/2019/05/22/VCpVns.png)
![VCpAXj.png](https://s2.ax1x.com/2019/05/22/VCpAXj.png)
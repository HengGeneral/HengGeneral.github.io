---
layout: post
title: Wireshark 工具
tags:  [网络]
categories: [工具]
author: liheng
excerpt: "网络封包工具"
---

### TCP Header Format

TCP头格式(header format)

    ![TCP-4](/images/network/tcp/tcp-header-format.png)

其中,

Source Port: 指定发送端口号

Destination Port: 指定接收端口号

Sequence Number, 有两个作用:

1. 若SYN标志位置为1, 表示这是最初的包的序号(从下图建立连接过程中可以看出, SYN=1 Seq=0); 同时,在该包来自服务端的确认信息中ack序号为seq+1
(从下图中建立连接过程中可以看出, SYN=1 ACK=1 Seq=0 Ack=1); 注: ACK: 全部大小表示标志; Ack: 首字母大写或者都不大写表示确认序号;

2. 如果SYN标志位置为0, 表示tcp包中data区第一个字节的序列标识;

Acknowledgment Number:
如果ACK标志位置顶为1, 该值表示希望另外一方下次发送包的序列号。如果之前发送过包的话,也是对之前数据接收的确认。
同时,每个端(client/server)发送的第一个ACK也是对另一个端初始序列号的确认。

Data Offset: 指定TCP头的字节数。头(header)最小是20 bytes, 最多是60 bytes, 即头(header)中的options字段最多40 bytes。

Flags: 包含9个位标识, 用到的如下:

1. ACK – 表示Acknowledgment Number很重要。除了发送端发送的SYN包以外, 其他的包都将该标志位设置1;
2. SYN (1 bit) – 同步序列号。只有sender/receiver发送的第一个包中改标志位才设为1。 
3. FIN (1 bit) – 表示发送者不会再有数据过来了

Window Size: 用于滑动窗口中, 表示接收端希望接收到(尚未获得确认消息)的字节数

Checksum: 用于header和data的数据校验,以检查数据传输过程中是否有错误。

### wireshark 学习TCP连接

打开wireshark, 打开浏览器, 输入www.baidu.com;

打开terminal, ping一下获取相应的主机host(这里是220.181.112.244), 如下:

    ![TCP-0](/images/network/tcp/tcp-ping-baidu.png)

在wireshark过滤工作栏中添加过滤器, 只显示与220.181.112.244的数据交互信息, 如下:

```
ip.dst==220.181.112.244 or ip.src ==220.181.112.244
```


1.第一次握手, 数据包

    ![TCP-1](/images/network/tcp/wireshark-tcp-1.png)


2.第二次握手, 数据包

    ![TCP-2](/images/network/tcp/wireshark-tcp-2.png)


3.第三次握手, 数据包

    ![TCP-3](/images/network/tcp/wireshark-tcp-3.png)

## 参考文献:

1.http://www.freesoft.org/CIE/Course/Section4/8.htm
2.https://nmap.org/book/tcpip-ref.html
3.http://coolshell.cn/articles/11564.html
4.https://en.wikipedia.org/wiki/Transmission_Control_Protocol
5.https://www.wireshark.org/docs/wsug_html_chunked/ChUsePacketBytesPaneSection.html
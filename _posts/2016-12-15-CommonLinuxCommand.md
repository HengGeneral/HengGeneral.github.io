---
layout: post
title: LINUX-常用的命令
tags:  [LINUX]
categories: [LINUX]
author: liheng
excerpt: "常用的linux命令记录"
---
### 查找类命令

#### du

功能介绍: 对文件和目录磁盘使用的空间的查看

使用场景:

1.  磁盘满了,要找到哪个目录的日志过多等

    ```
        du -h --max-depth=1
    ```

#### grep

功能介绍: 一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

使用场景:

1.  查找某个文本, 并把相应行打印出来;

    ```
       grep error ./catalina.out
    ```

2.  查找某个文本, 不区分大小写, 并将相应行打印出来;

    ```
       grep -i error ./catalina.out
    ```

3.  查找某个文本, 并将该行的前三行和后五行打印出来;

    ```
       grep -B 3 -A 5 error ./catalina.out 
    ```

4.  查找某个文本, 将该文本高亮显示, 同时打印出该行;

    ```
       grep --color=auto error ./catalina.out
    ```

5.  过滤某个文本, 将除了某文本行之外的其它行打印出来;

    ```
        grep -v error ./catalina.out ##过滤error文本
        
        grep -v ^$ ./catalina.out ##过滤空行
        
    ```

6.  还可以与正则表达式结合使用, 使用之前注意查看该unix系统相应的参数。

#### awk

功能介绍: 把文件逐行读入，以空格(或者其他指定字符)为分隔符, 将每行切片，切开的部分再进行各种分析处理。

使用场景:

1.  根据日志每行的时间进行切片, 用于统计qps

    ```
        awk -F "." '{print $1}' ./catalina.out
    ```

#### uniq

功能介绍: 用于过滤重复的行

使用场景:

1.  计算qps

    ```
        awk -F "." '{print $1}' ./catalina.out | uniq -c 
    ```

2.  仅显示文件中不重复的行

    ```
        uniq -u ./test.txt
    ```

3.  仅显示文件中重复的行

    ```
        uniq -d ./test.txt
    ```

### 压缩类

#### zip/unzip

功能介绍: 压缩/列出/解压zip压缩文件

使用场景: 

1.  解压缩zip文件

    ```
      unzip ./test.zip
    ```

#### tar

功能介绍: 操作tar压缩文件

使用场景:

1.  解压缩tar.gz文件

    ```
    tar -xvf test.tar.gz
    ```
    
    其中: -x 表示解压缩(Extract to disk from the archive), -v 表示解压缩后的文件名(produce verbose output), -f 后面跟
    要解压缩的文件名(read the archive from)
    
### 系统配置类

#### uname

功能介绍:
查看操作系统的相关信息

使用场景:

1.  输出该操作系统版本(-r), 节点名称(-n)等信息

    ```
        uname -a
    ```

#### sz/rz/scp

功能介绍: unix 和 windows 文件传输的命令行工具

使用场景:

1.  从服务器上下载文件

    ```
        sz test.txt
    ```

2.  把本地文件上传到服务器

    ```
        scp file user@host:file
    ```
    
### 网络类
    
#### curl

功能介绍: 用于c/s之间传输数据

使用场景:

1.  测试服务启动情况, 比如判断es集群启动情况

    ```
        curl https://XX.XX.XX.XX:9200/?pretty
    ```
    
#### tcpdump

功能介绍: 打印网络数据包

使用场景:

1.  测试与指定服务器列表的通信

    ```
        tcpdump host host1 or host2 or host3
    ```
    
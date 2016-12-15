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

1. 磁盘满了,要找到哪个目录的日志过多等

```
    du -h --max-depth=1
```

#### grep

功能介绍: 一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

使用场景:

1. 查找某个文本, 并把相应行打印出来;

```
   grep error ./catalina.out
```

2. 查找某个文本, 不区分大小写, 并将相应行打印出来;

```
   grep -i error ./catalina.out
```

3. 查找某个文本, 并将该行的前三行和后五行打印出来;

```
   grep -B 3 -A 5 error ./catalina.out 
```

4. 查找某个文本, 将该文本高亮显示, 同时打印出该行;

```
   grep --color=auto error ./catalina.out
```

5. 还可以与正则表达式结合使用, 使用之前注意查看该unix系统相应的参数。

### 系统配置类

#### uname

功能介绍:
查看操作系统的相关信息

使用场景:

1. 输出该操作系统版本(-r), 节点名称(-n)等信息

```
    uname -a
``
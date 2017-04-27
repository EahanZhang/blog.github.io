# Apache Flume学习笔记

标签（空格分隔）： 分布式 日志聚合系统

---

## 什么是Flume?
Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，目前已经是Apache的一个子项目其设计的原理也是基于将数据流，如日志数据从各种网站服务器上汇集起来存储到HDFS，HBase等集中存储器中。其结构如下图所示：
![](http://images2015.cnblogs.com/blog/539316/201607/539316-20160710192339483-1093743457.jpg)

## Flume OG 
Flume OG : Flume Original Generation, 初代Flume。由三种角色构成：代理点Agent, 收集节点Collector, 主节点Master。
- Agent：从各个数据源收集日志数据， 将收集到的数据几种到Collector，然后由Collector汇总存入HDFS。
- Master：负责管理Agent，Collector的活动。
- Agent、Collector 都称为 node，node 的角色根据配置的不同分为 logical node（逻辑节点）、physical node（物理节点）。对 logical nodes 和 physical nodes 的区分、配置、使用一直以来都是使用者最头疼的地方。
- agent、collector由Source、Sink组成，当前节点的数据是从Source传送到Sink的。
![](http://upload-images.jianshu.io/upload_images/2838375-b134e4c6962417d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Flume NG 
Flume NG : Flume New Generation.
- NG 只用一种角色节点： 代理点（agent）
- 没有collector和master节点
- 去除了physical nodes和logical nodes的概念和相关内容。
- NG agent由source、sink和channel组成。
![](http://upload-images.jianshu.io/upload_images/2838375-97e9d808f92d23a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 安装 Flume NG
### 版本
编辑此文时当前最新Flume版本为1.7.0，[点击此处下载apache-flume-1.7.0-bin.tar.gz](https://flume.apache.org/download.html)

### 系统要求
1. Java版本不低于1.7
2. 足够的内存
3. 足够的硬盘空间
4. 拥有文件夹的读写权限

### 安装Java
1. 从[Oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)上下载JDK安装包，解压安装：
`$sudo mkdir /usr/java`
`$sudo tar zxvf jdk-8u121-linux-x64.tar.gz -C /usr/java`

2. 配置java环境变量
`$sudo vi /etc/profile`
添加如下内容：
`# set java environment`
`JAVA_HOME=/usr/java/jdk1.8.0_121`
`JRE_HOME=/usr/java/jdk1.8.0_121/jre`
`CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib`
`PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin`
`export JAVA_HOME JRE_HOME CLASS_PATH PATH`

3. 让修改生效
`$source /etc/profile`

### 安装Flume
将下载好的二进制包解压缩并改名:
`$sudo tar zxvf apache-flume-1.7.0-bin.tar.gz -C /usr/local/`
`$cd /usr/local`
`$sudo mv apache-flume-1.7.0-bin/ flume/`

### 配置Flume
在flume目录下创建一个example.conf的文件
`cd flume`
`sudo vim example.conf`
在该文件中添加以下内容：
```
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
添加完成后，保存并退出

### 运行Flume
配置完成后使用下面的命令运行Flume:
`$sudo /usr/local/flume/bin/flume-ng agent --conf conf --conf-file /usr/local/flume/example.conf --name a1 -Dflume.root.logger=INFO,console`
运行上述命令后打开一个新的终端来测试Flume
`$telnet localhost 44444`
`Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
Hello world! <ENTER>
OK`
运行Flume的终端上将会输出信息：
`17/04/17 15:32:19 INFO source.NetcatSource: Source starting`
`17/04/17 15:32:19 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]`
`17/04/17 15:32:34 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }`
Flume配置、运行成功。

## 参考
> http://www.jianshu.com/p/5fbf57980b87
> http://www.cnblogs.com/gongxijun/p/5656778.html
> http://www.jianshu.com/p/2f74183fe727






---
layout: post
title: "centos中 flume+kafka 安装、配置和使用"
date: 2017-04-27
---

## 简介
flume - flume是一个可高效收集数据并将数据移动到集中式数据存储中的分布式系统。
kafka - kafka是一个分布式的消息系统，其优点是高吞吐率和可水平扩展，并且在保持高吞吐率的同时，时间复杂度为O(1)。
两者的结合使用可以实现的功能 - 使用flume监听日志文件并且将监听到的信息传输给kafka，kafka将接受到的消息做进一步处理，并将处理结果用到需要的地方。

## Java 安装
flume 和 kafka 都需要Java环境，因此需要首先安装Java， Java版本应不低于1.7，此处使用的版本是Java 1.8。
Java的安装方法请见[centos 安装java, tomcat和mysql](https://www.zybuluo.com/yhzhang/note/722738)

## flume 安装
编辑此文时当前最新Flume版本为1.7.0.
[点击此处下载apache-flume-1.7.0-bin.tar.gz](https://flume.apache.org/download.html)
1. 将下载后得到的二进制包解压
```
$sudo tar zxvf apache-flume-1.7.0-bin.tar.gz -C /usr/local/
```

2. 修改文件夹名称
```
$cd /usr/local
$sudo mv ./apache-flume-1.7.0-bin ./flume
```

3. 配置环境变量
```
$sudo vim /etc/profile
```

在文件末尾添加以下内容
```
# set flume environment
export FLUME_HOME=/usr/local/flume
export FLUME_CONF_DIR=$FLUME_HOME/conf
export PATH=$PATH:$FLUME_HOME/bin
```

保存并退出
4. 使配置生效
```
$source /etc/profile
```

安装、配置完成。

## kafka 安装
1. 去[官网](http://kafka.apache.org/downloads.html)下载安装包。编写本文时kafka最新版本为0.10.2.0，此处下载的是kafka_2.11-0.10.2.0.tgz。
2. 解压缩
```
$sudo tar zxvf kafka_2.11-0.10.2.0.tgz -C /usr/local/
```

3. 改名
```
$cd /usr/local
$sudo mv ./kafka_2.11-0.10.2.0 ./kafka
```

安装完成。


## flume + kafka 使用
1. 开启zookeeper server。因为kafka使用zookeeper，所以首先需要开启zookeeper server
```
$cd /usr/local/kafka
$./bin/zookeeper-server-start.sh $./config/zookeeper.properties
```

2. 开启kafka server
```
$./bin/kakfa-server-start.sh $./config/server.propertires
```

3. 构造kafka topic
```
$./bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic flume-kafka-topic
```

4. 修改flume配置文件
```
$cd /usr/local/flume
$sudo vim ./conf/flume-kafka.conf
```

添加以下内容
```
# set name 
kafka.sources = source_log
kafka.channels = channel_log
kafka.sinks = sink_log

# configure the sources
kafka.sources.source_log.type = TAILDIR

kafka.sources.source_log.filegroups = f1
kafka.sources.source_log.filegroups.f1 = /home/hadoop/data/log/.*.log

kafka.sources.source_log.skipToEnd = True
kafka.sources.source_log.positionFile = /home/hadoop/data/taildir_position.json
kafka.sources.source_log.batchSize = 1000
kafka.sources.source_log.channels = channel_log

# configure the sinks
kafka.sinks.sink_log.type = org.apache.flume.sink.kafka.KafkaSink
kafka.sinks.sink_log.kafka.topic = flume-kafka-topic
kafka.sinks.sink_log.kafka.bootstrap.servers = localhost:9092
kafka.sinks.sink_log.kafka.flumeBatchSize = 20
kafka.sinks.sink_log.kafka.producer.acks = 1
kafka.sinks.sink_log.kafka.producer.lingers.ms = 1
kafka.sinks.sink_log.kafka.compression.type = snappy
kafka.sinks.sink_log.channel = channel_log

# configure the channels
kafka.channels.channel_log.type = memory
kafka.channels.channel_log.capacity = 1000
kafka.channels.channel_log.transactionCapacity = 100
```

其中
- line 10 和 line 13中的文件夹可以改为你自己的目录，需要提前创建好。
- line 20 中的localhost可以改为指定IP地址
- line 28 中的type可以改为file或其他能存储在硬盘中的选项。
5. 运行
```
cd /usr/local/flume
```

执行命令：
```
$./bin/flume-ng agent --conf ./conf --conf-file ./conf/flume-kafka.conf --name kafka -Dflume.root.logger=DEBUG,console
```

6. 测试
运行完成后，可以在line 10中对应的目录下创建一个日志文件（如4.25.log），并往里写入一些信息（如echo "hello world" >> 4.25.log），此时将会从kafka的consumer中得到输出：
打开一个新的终端，输入以下命令：
```
$cd /usr/local/kafka
$./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flume-kafka-topic
```

此时将会从终端显示输入到日志文件中的信息，并且每当日志文件内容发生增长，此终端上就会显示其增长的内容。





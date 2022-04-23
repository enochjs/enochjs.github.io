---
title: rocketmq
date: 2022-04-23 16:28:48
categories:
  - nodejs
tags:
---

### 记一次使用 docker 安装 rocketMQ,记录

#### [基本概念](https://github.com/apache/rocketmq/blob/master/docs/cn/concept.md)

1. 消息模型（Message Model）

```dotnetcli
RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。
Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。
```

2. 消息生产者（Producer）

```dotnetcli
负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。
RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。同步和异步方式均需要Broker返回确认信息，单向发送不需要。
```

3. 消息消费者（Consumer）

```dotnetcli
负责消费消息，一般是后台系统负责异步消费。一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。
从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。
```

4. 主题（Topic）

```dotnetcli
表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。
```

5.  代理服务器（Broker Server）

```dotnetcli
消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。
代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。
```

6. 名字服务（Name Server）

```dotnetcli
名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。
多个Namesrv实例组成集群，但相互独立，没有信息交换。
```

#### [架构设计](https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md)

设计图
![](rocketmq-architecture-1.jpg)

#### 如何使用 docker 安装

1. 安装 name-server

```bash
# ${workspace} 换成你想存储的目录，如 /Users/enochjs/docker
docker pull rocketmqinc/rocketmq
docker run -d --restart=always --name rmqnamesrv -p 9876:9876 -v ${workspace}/data/rocketmq/log:/root/logs -v ${workspace}/data/rocketmq/store:/root/store -e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq sh mqnamesrv
```

2. 安装 broker

   - 创建 broker 存储节点

   ```bash
    # ${workspace} 换成你想存储的目录，如 /Users/enochjs/docker
    # ${ip} 你电脑的ip 如：10.0.116.233
    mkdir -p  ${workspace}/data/rocketmq/data/broker/logs   ${workspace}/data/rocketmq/data/broker/store ${workspace}/data/rocketmq/conf
   ```

   - 编辑 broker.conf

   ```bash
   # 所属集群名称，如果节点较多可以配置多个
   brokerClusterName = DefaultCluster
   #broker名称，master和slave使用相同的名称，表明他们的主从关系
   brokerName = broker-a
   #0表示Master，大于0表示不同的slave
   brokerId = 0
   #表示几点做消息删除动作，默认是凌晨4点
   deleteWhen = 04
   #在磁盘上保留消息的时长，单位是小时
   fileReservedTime = 48
   #有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
   brokerRole = ASYNC_MASTER
   #刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
   flushDiskType = ASYNC_FLUSH
   #name-server 地址
   namesrvAddr = ${ip}:9876
   #设置 broker 节点所在服务器的 ip 地址
   brokerIP1 = ${ip}
   ```

   - 启动 broker

   ```bash
   # ${workspace} 换成你想存储的目录，如 /Users/enochjs/docker
   # ${ip} 你电脑的ip 如：10.0.116.233
   docker run -d  --restart=always --name rmqbroker --link rmqnamesrv:namesrv -p 10911:10911 -p 10909:10909 -v ${workspace}/data/rocketmq/data/broker/logs:/root/logs -v  ${workspace}/data/rocketmq/data/broker/store:/root/store -v ${workspace}/data/rocketmq/conf/broker.conf:/opt/rocketmq/conf/broker.conf -e "NAMESRV_ADDR=${ip}:9876" -e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq sh mqbroker -c /opt/rocketmq/conf/broker.conf
   ```

3. 安装控制台界面, 访问 http://${ip}:8888,即可
   ```bash
    docker run -d --restart=always --name rmqadmin -e "JAVA_OPTS=-Drocketmq.namesrv.addr=${ip}:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -p 8888:8080 pangliang/rocketmq-console-ng
   ```

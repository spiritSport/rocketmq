

## 默认生成者类的核心类
![image](ClientDefaultProducer核心类图.png)

## 启动流程
![image](ClientDefaultProducer启动流程.png)

3.将clientName设置为本程序的pid，为其他程序读取clintId做准备

7.启动几个定时任务

时间单位默认毫秒

| 方法名                             | 初始延时  | 时间间隔  |                                           | 功能    | 备注 |
|:-----------------------------------|:----------|:----------|:------------------------------------------|:--------|:----|
| fetchNameServerAddr                | 1000 * 10 | 2分钟     | 取name server的地址                        | 无效代码 |     |
| updateTopicRouteInfoFromNameServer | 10        | 1000 * 30 | 从                                        |         |     |
| cleanOfflineBroker                 | 1000      | 1000 * 30 | 远端拉取topi生产者和消费者信息并同步到本地的 |         |     |
| sendHeartbeatToAllBrokerWithLock   | 1000      | 1000 * 30 |                                           |         |     |
| persistAllConsumerOffset           | 1000 * 10 | 1000 * 5  |                                           |         |     |
| adjustThreadPool                   | 1分钟     | 1分钟     |                                           |         |     |


fetchNameServerAddr
从1个http://jmenv.tbsite.net:8080/rocketmq/nsaddr的地址获取name server，但是实际上该地址是无法访问的


updateTopicRouteInfoFromNameServer
从MQClientInstance类的producerTable，consumerTable取到topic列表，
远端拉取topi生产者和消费者信息,如果和本地数据不一致，那么同步到本地的MQClientInstance类的
brokerAddrTablebrokerAddrTable，producerTable，consumerTable成员

![image](../../diagram/classDiagram/MQClientInstanceUpdateTopicRouteInfoFromNameServer.png)


![image](../../diagram/classDiagram/updateTopicRouteInfoFromNameServer.png)


### cleanOfflineBroker
纯本地数据处理
brokerAddrTable中的broker不在topicRouteTable中的话，删除这个broker
topicRouteTable会通过心跳定时获取到更新

### sendHeartbeatToAllBrokerWithLock
向brokerAddrTable中的全部broker发送心跳数据，具体数据结构如下图
![image](../../diagram/classDiagram/HeartbeatData.png)
返回版本号，并将版本号放入 brokerVersionTable

### persistAllConsumerOffset
将消费者offset持久化
push模式和pull模式持久方法不一样，前者是提交数据到broker，后者是存储在本地文件

### adjustThreadPool
看起来像是调整consumer的线程数，实际代码全部注释，未执行实际操作

# 数据发送和接收
发送和接收都通过netty实现，客户端是netty client 服务器borker是netty Server
## 数据发送流程
TODO

## 数据接收流程
TODO

## [clientId生成规则](clientId生成规则.md)


****

### 一些启动相关的疑问
DefaultMQProducerImpl 的start方法调用2次，好像只是为了发送2次心跳到全部broker
brokerVersion是干什么用的

发送数据流程

发送普通消息
发送定时消息
发送事务消息
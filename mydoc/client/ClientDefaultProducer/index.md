

## 默认生成者类的核心类
![image](ClientDefaultProducer核心类图.png)

启动流程
![image](ClientDefaultProducer启动流程.png)
DefaultMQProducerImpl 的start方法调用2次，好像只是为了发送2次心跳到全部broker
发送数据流程

发送普通消息
发送定时消息
发送事务消息

[clientId生成规则](clientId生成规则.md)


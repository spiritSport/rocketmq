
##
DefaultMessageStore 入口
AbstractPluginMessageStore 这个类未启用

## store功能描述

存储commitLog，consumeQueue，cosumeQueueExt，index信息到本地硬盘
主从数据同步 主从管理 延时消息 BrokerState管理


## store模块划分

| 模块名称      | 功能描述                    | 备注         |
|:-------------|:---------------------------|:-------------|
| 核心模块      | 写入消息、读取消息、查询消息 |              |
| 文件管理      |                            | MappedFile等 |
| commitLog    |                            |              |
| consumeQueue |                            |              |
| index        |                            |              |
| ha           |                            |              |
| 延时队列      |                            |              |
| 状态管理      |                            |              |
| 消息过滤      |                            |              |


## DefaultMessageStore定时任务

|                                    |          |           |                    |
|:-----------------------------------|:---------|:----------|:-------------------|
| cleanFilesPeriodically             | 1000 * 6 | 1000 * 10 |                    |
| checkSelf                          | 1分钟    | 10分钟    |                    |
| string2FileNotSafe                 | 1秒      | 1秒       |                    |
| ReputMessageService.doReput        | 0        | 1         | ServiceThread      |
| FlushConsumeQueueService.doFlush   | 0        | 10        | ServiceThread      |
| StoreStatsService.run              |          | 1000      |                    |
| HAService.AcceptSocketService.run  |          |           |                    |
| HAService.GroupTransferService.run |          |           |                    |
| HAService.HAClient.run             |          |           |                    |
| ScheduleMessageService.run         |          |           | timer触发事件不固定 |

##CommitLog的定时任务

|                             |   |     |   |
|:----------------------------|:--|:----|:--|
| CommitRealTimeService.run   |   | 200 |   |
| FlushRealTimeService.run    |   | 500 |   |
| GroupCommitService.doCommit |   | 10  |   |


### 二次写入

DefaultMessageStore.ReputMessageService.doReput
这个方法1毫秒触发1次，循环执行。
执行DefaultMessageStore.this.doDispatch(dispatchRequest);
触发消息到达事件（部分等待消息的channel可以获取新的消息）
StoreStatsService.singlePutMessageTopicTimesTotal+1
StoreStatsService.singlePutMessageTopicSizeTotal+1

ReputMessageService本身是个ServerThread


FlushConsumeQueueService 基于ServerThread
刷新本地consumeQueueTable、consumeQueue和consumeQueueExt，刷新刷新DefaultMessageStore.checkpoint.LogicsMsgTimestamp


## 硬盘存储数据

### commitLog

分多个文件存储，每个文件内以message的形式存储
![image](../visio/commitLog.png)

messge分2种MessageExtBrokerInner和MessageExtBatch

MessageExtBrokerInner存储结构如下

|    名称       | 字节数   | 描述   |
|:----------|:---|:---|
| TOTALSIZE | 4  |    |
| MAGICCODE | 4   |    |
| BODYCRC  |  4  |  crc32 算法，对body整体做摘要   |
| QUEUEID |  4  |    |
| FLAG | 4  |    |
|QUEUEOFFSET | 8  |    |
|PHYSICALOFFSET | 8  |    |
|SYSFLAG |    4|    |
|BORNTIMESTAMP | 8   |    |
|BORNHOST |    8| ？   |
|STORETIMESTAMP |  8  |    |
|STOREHOSTADDRESS |  8  |    |
| RECONSUMETIMES | 4 |    |
| Prepared Transaction Offset  |  8  |    |
|    BODY       |  4+body长度  |   前4位记录body的长度，后边是body的正文 |
|TOPIC |  1+Topic的长度  | 前1位表示topic字符串的长度，后边的是topic的值（UTF-8）   |
|PROPERTIES| 2+propertiesData的长度   |前2位是propertiesData，后边是propertiesData的正文    |

MessageExtBatch存储结构如下

| 名称       | 字节数 | 描述                                                    |
|:-----------|:------|:-------------------------------------------------------|
| TOTALSIZE  | 4     |                                                        |
| MAGICCODE  | 4     |                                                        |
| BODYCRC    | 4     | crc32 算法，对body整体做摘要                                      |
| FLAG       | 4     |                                                        |
| BODY       |       | 1个复合结构， 内容是MessageExtBrokerInner的列表，按顺序写 |
| properties |       |                                                        |

### consumeQueue
单个元素总长20字节

| 名称     | 字节数 | 描述               |                           |
|:---------|:------|:-------------------|:--------------------------|
| offset   | 8     | commitLog中的偏移量 |                           |
| size     | 4     | 这条消息的size      |                           |
| tagsCode | 8     | 消息的tagCodes     | TODO 这里的生成规则没搞清楚 |


### consumeQueueExt

TODO 没看懂

| 名称         | 字节数 | 描述 |                           |
|:-------------|:------|:----|:--------------------------|
| size         | 2     |     |                           |
| tagsCode     | 8     |     |                           |
| msgStoreTime | 8     |     | TODO 这里的生成规则没搞清楚 |
| bitMapSize   | 2     |     |                           |
| filterBitMap | 不固定 |     |                           |

### Index

前40byte是 indexheader，按如下规则

    private static int beginTimestampIndex = 0;
    private static int endTimestampIndex = 8;
    private static int beginPhyoffsetIndex = 16;
    private static int endPhyoffsetIndex = 24;
    private static int hashSlotcountIndex = 32;
    private static int indexCountIndex = 36;

hashSlotNum * hashSlotSize 是hash index的位置，每4字节是1个值，标注index的位置

之后是index的内容区，1个元素20位
keyHash 4字节
phyOffset 8字节
timeDiff 4字节
slotValue 4 字节 索引的插入序号，从1开始+1，累计变为负数的时候，重新从1开始

### checkPoint

physicMsgTimestamp 8字节
logicsMsgTimestamp 8字节
indexMsgTimestamp 8字节

### DLedgerCommitLog
TODO 开启DLegerCommitLog的模式下才用到，非默认模式


## 重要的流程
数据插入（同步刷盘也存在丢数据）

数据读取

数据查询


## 几个机制的实现
ha

定时任务实现

数据淘汰机制

## 问题
缓存机制

硬盘忙如何监控


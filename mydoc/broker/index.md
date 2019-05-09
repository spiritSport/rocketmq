
## broker功能
和client nameserver通讯

消息写入、读取、查询

## BrokerController定时任务

时间单位默认毫秒

| 方法                                   | 启动延时   | 运行间隔            | 备注 |    |    |
|:---------------------------------------|:-----------|:--------------------|:----|:---|:---|
| BrokerStats.record()                   |            | 1000 * 60 * 60 * 24 |     |    |    |
| ConsumerManager.persist()              | 1000  * 10 | 1000 * 5            |     |    |    |
| ConsumerFilterManager.persist()        | 1000 * 10  | 1000 * 10           |     |    |    |
|                                        |            |                     |     |    |    |
| BrokerController.protectBroker()       | 3分钟      | 3分钟               |     |    |    |
| BrokerController.this.printWaterMark() | 10秒       | 1秒                 |     |    |    |
| MessageStore.dispatchBehindBytes()     | 100 * 10   | 1000 * 60           |     |    |    |
| BrokerOuterAPI.fetchNameServerAddr()   | 1000 * 10            |  10000 * 60                    |     |    |    |
|            BrokerController.this.printMasterAndSlaveDiff()                            |    1000 *10         |    1000 * 60                  |     |    |    |
|                                        |            |                     |     |    |    |

## 客户端请求数据，broker无新数据时如何处理

PullRequestHoldService
这个是为了拉不到数据的请求设计，客户端请求数据时，如果数据找不到，那么先放到
PullRequestHoldService.pullRequestTable里，key是topic@queueId
当生产者发到新的数据时，判断是否是pullRequestTable里的queue，
是的话，执行后续的pullRequest操作

也通过轮训的方式
调用checkHoldRequest方法，通过offset对比的方法，判断是否需要返回数据

## slave读数据
第一遍pullRequest，主broker返回1个建议读取数据的borkerid，消费者在下次pull的时候，就按新的来取

提交offset还是提交到主borker，否则会不一致
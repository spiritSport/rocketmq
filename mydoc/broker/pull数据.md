
broker有个   ExecutorService pullMessageExecutor 负责使用线程池方式获取数据

```java

                this.brokerConfig.getPullMessageThreadPoolNums(),
                this.brokerConfig.getPullMessageThreadPoolNums(),
                1000 * 60,
                TimeUnit.MILLISECONDS,
                this.pullThreadPoolQueue,
                new ThreadFactoryImpl("PullMessageThread_"));

```
 线程池的队列使用blockqueue

 ```java
 BlockingQueue<Runnable> pullThreadPoolQueue
```



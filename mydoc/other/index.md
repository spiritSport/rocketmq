
reblance如何实现?


从开始 一直到ConsumeMessageConcurrentlyService都是单线程，这里通过  ThreadPoolExecutor consumeExecutor
实现了多线程



NettyClientHandler.channelRead0 netty客户端收到消息的入口

NettyRemotingClient.NettyClientHandler 实现netty的handler 继承netty的 SimpleChannelInboundHandler

netty channel.writeAndFlush 负责发出数据

NettyClientHandler.channelRead0或类似方法，实现读数据



NettyRemotingClient.invokeSync时，先创建1个Channel，然后通过chnnel flush 提交数据，同时设置好发送成功的回调方法


RemotingCommand 请求和响应都是这个数据结构

RequestTask 是borker 专用功能，client没关系


PullRequestHoldService 初步认为是将拉取数据没拉到的pullreqest放到1个map，缓存一下，然后再慢慢拉。
如果等待时间超过了超时时间，那么执行pullmessage操作






<!--| adsf | sdf | sadf |-->
<!--|:-----|:----|:-----|-->
<!--| 123  | 123 | asdf |-->


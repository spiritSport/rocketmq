

# RemotingClient
通过netty进行数据请求和响应，在netty体系中是client角色。需要主动发起请求，才能收到数据。
目前每次都是创建1个channel进行请求数据，完成后销毁。不具备长连接


![image](../../diagram/classDiagram/NettyRemotingClient.png)


## oneWay请求的流程

![image](../../diagram/classDiagram/nvokeOneway.png)

##  sync请求的流程

![image](../../diagram/classDiagram/invokeSync.png)

## async请求的流程

![image](../../diagram/classDiagram/invokeAsync.png)
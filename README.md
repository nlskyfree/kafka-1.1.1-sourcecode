Apache Kafka 1.1.1 源码注释版
=================
## 源码编译调试

1. 编译依赖：Java8 + gradle-4.8.1亲测编译成功，gradle高版本会缺少一些插件
2. 执行gradle idea成功后，导入idea
3. 在idea应用启动配置中配置Program arguments，指定server.properties配置文件路径
4. 修改server.properties，指定kafka数据路径与zk路径，即可启动调试


## 源码注释进度
#### 1. 网络层注释：已完成 
```
SocketServer                           //kafka网络层的封装
   |-- Acceptor                        //Acceptor线程的封装
   |-- Processor                       //Processor线程的封装
Selector                               //对java selector的封装，封装了核心的poll，selectionkeys的遍历，事件的注册等操作
KafkaChannel                           //对java SocketChannel的封装，封装是实际的读写IO操作
TransportLayer                         //对KafkaChannel屏蔽了底层是使用Plaintext不加密通信还是ssl加密通信
RequestChannel                         //和API层通信的通道层，封装了和API层通信的Request、Response以及相应的通信队列
  |-- Request                          //传递给API层的Requst
  |-- Response                         //API层返回的Response
```   
#### 2. API层注释：已完成
```
KafkaRequestHandler
   |-- run()                           //从全局阻塞队列获取网络层组装完的Request，并调用KafkaApi的handle方法
KafkaApis
   |-- handle()                        //所有请求的入口，根据ApiKeys分发请求
```
##### 2.1 ApiKey.PRODUCE请求：进行中
```

```
#### 3. ReplicaManager：进行中
#### 4. Log：进行中
```
```
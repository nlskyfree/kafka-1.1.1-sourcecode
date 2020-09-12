Apache Kafka 1.1.1 源码注释版
=================
## 源码编译调试

1. 编译依赖：Java8 + gradle-4.8.1亲测编译成功，gradle高版本会缺少一些插件
2. 执行gradle idea成功后，导入idea
3. 在idea应用启动配置中配置Program arguments，指定server.properties配置文件路径
4. 修改server.properties，指定kafka数据路径与zk路径，即可启动调试


## 源码注释进度
### 一、服务端
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
##### 2.1 ApiKey.PRODUCE请求：已完成
TODO：1.没有加上事务、幂等处理的注释  2.延迟任务，时间轮 3. 索引文件的逻辑没有详细注释 4、权限设计，这几块也没有注释，比较复杂，可放入单独章节 
```
KafkaApis
   |--handleProduceRequest()           //API入口，主要对session进行权限校验，调用replicaManager.appendRecords写入数据
   |--sendResponseCallback()           //数据写入完成后的回调函数，封装Response，发送到网络层的响应队列中
   |--replicaManager.appendRecords()
ReplicaManager
   |--appendRecords()                  //执行数据写入逻辑
      |--appendToLocalLog()            //遍历每个TopicPartition，若当前Broker为leader，执行写入逻辑
      |--delayedProduceRequestRequired() //根据ack判断是否需要等待副本同步
      |--tryCompleteElseWatch()        //若需要等待副本同步，提交一个延迟任务
   |--responseCallback()               //若无需等待副本同步，执行回调函数，返回response，实际调用了sendResponseCallback
Partition                              //partition的逻辑封装
   |--appendRecordsToLeader()          
      |--inSyncSize < minIsr && requiredAcks == -1 //MINISR校验，校验失败抛出异常
      |--log.appendAsLeader()
      |--replicaManager.tryCompleteDelayedFetch       //有些fetch请求可能正在等待，若有新数据写入，可尝试唤醒fetch请求
      |--maybeIncrementLeaderHW        //数据写入完成，HW可能发生变化，比如只有一个leader在ISR中
Log                                    //一个log目录，一系列segment的封装
   |--appendAsLeader()                 //leader写入时需执行更新batch的offset逻辑，实际是对append的封装
      |--append()                      //数据写入逻辑
         |--analyzeAndValidateRecords()//Batch size校验，CRC校验计算，判断客户端、服务端压缩算法
         |--LogValidator.validateMessagesAndAssignOffsets() //校验 + 消息格式转换成Topic对应的消息版本 + 压缩转换成broker的压缩方式
         |--maybeRoll()                //判断是否滚动segment，函数永远返回active segment
         |--segment.append()
         |--updateLogEndOffset()       //更新LEO
         |--updateFirstUnstableOffset  //更新LSO   
LogSegment                             //Segment的逻辑封装
   |--log.append()                     //这里的log实际是FileRecords
   |--offsetIndex.append()             //稀疏索引构建，到达固定间隔才会调用，写入offset与log文件物理偏移的映射
   |--timeIndex.maybeAppend()          //稀疏索引构建，到达固定间隔才会调用，写入时间戳与log文件物理偏移的映射
FileRecords                            //这个类就是底层文件的封装，封装了File、FileChannel                 
   |--append()                         //调用FileChannel的write方法写入数据
```
##### 2.1 ApiKey.FETCH请求：进行中
```

```
#### 3. ReplicaManager：进行中
#### 4. Log：进行中
```
```
### 一、客户端
#### 1. 生产者
##### 1.1 Producer线程：进行中
##### 1.2 Sender线程
##### 1.3 网络层
##### 1.4 RecordAccumulator
##### 1.5 BufferPool
#### 2. 消费者

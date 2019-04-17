[TOC]

# Hadoop hdfs

## 简介

- **特点**

1. 支持分布式的计算
2. 分而治之，并行计算
3. 计算向数据移动

- **重要组件**

1. hdfs 分布式存储系统主要用于支持数据计算。
2. mapreduce 分布式计算
3. yarn 资源管理及调度框架

## hdfs1.X

### hdfs的存储模型

- 是以`字节`为单位进行存储的
- 文件被线性的切割`block`块，`block`的属性有大小和偏移量
- 同一文件的`block`块的大小是一致的，不同文件之间可以不一致
- `block`的副本数量是可以配置的，一般建议其数量不要超过节点的数量
- 文件上传的时候可以设置`block`块的大小和副本数量
- 已经上传的文件的`block`块的副本数量可以调整但是大小不能改变
- 只支持一次写入多次读取，同一时刻只有一个写入者，可以`append`追加数据

### 架构模型

- **角色**

- ```
  namenode
  ```

  - 特点

  1. 基于内存存储，不会和磁盘进行交互

  - 功能

  1. 用于存储文件的元数据信息（文件的基本信息，block的位置信息，偏移量）
  2. 接收客户端的读写的请求
  3. 收集datanode的bock块的列表信息
  4. 其内部维护了一个虚拟的目录树结构。

  - 持久化

  1. NameNode的metadate信息在启动后会加载到内存
  2. metadata存储到磁盘文件名为”fsimage”
  3. Block的位置信息不会保存到fsimage
  4. edits记录对metadata的操作日志

- `datanode`

1. 用于存储数据(block),以文件的形式;
2. 同时也会存储block的元数据信息
3. 启动DataNode时向其汇报block的元数据信息
4. 与namenode保持心跳(3s),如果10分钟内没有接收到datanode的心跳信息，就认为其lost,并将其上的block块及copy到其他节点上，以维持block快的副本数量。

- `secondaryNamenode` hadoop1.X

1. 定期合并操作日志形成新的fsimage,进行元数据的持久化
2. 不是namenode的备份，不具有namenode的功能，它的主要工作是帮助NN合并edits log，减少NN启动时间。
3. 合并的时机
   - 根据配置文件设置的时间间隔fs.checkpoint.period 默认3600秒
   - 根据配置文件设置edits log大小 fs.checkpoint.size 规定edits文件的最大值默认是64MB

![合并edit形成fsimage的流程图](https://img-blog.csdnimg.cn/20181226171442648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

- **架构图** ![hdfs架构图](https://img-blog.csdnimg.cn/20181226163529238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

### block副本的放置策略

- 第一个副本：放置在上传文件的DN；如果是集群外提交，则随机挑选一台磁盘不太满，CPU不太忙的节点。
- 第二个副本：放置在于第一个副本不同的 机架的节点上。
- 第三个副本：与第二个副本相同机架的节点。
- 更多副本：随机节点

### 优缺点

- 优点
  - 高容错性：数据自动保存多个副本，副本丢失后自动恢复副本。
  - 适合批处理：移动计算而非数据，数据位置暴露给计算框架(block的偏移量)。
  - 适合大数据处理
  - 可构建在廉价的机器上
- 缺点
  - 不适合小文件的存取，占用namenode的大量内存，寻道时间大于读取时间；
  - 一个文件只有一个写入者，不支持修改，只支持append.

### hdfs的写流程

![hdfs的写流程](https://img-blog.csdnimg.cn/20181226171849305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

1. 客户端将文件进行切分成 block块,线性上传block
2. 向namenode获取datanode的列表，获取第一个block的节点信息（3副本，选择机制）
3. 验证完DN列表后和和DN通信：pipeline：C和1stDN有socket，1stDN和2edDN有socket。。。。
4. block传输结束后，DataNode向namenode汇报block信息，DN向Client汇报，Client向NN汇报完成
5. 获取下一个Block存放的DN列表
6. 最终Client汇报完成
7. NN会在写流程更新文件状态为可用

### hdfs的读流程

![hdfs读数据的流程](https://img-blog.csdnimg.cn/20181226190446478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

1. 客户端向namenode获取一部分block块的位置列表
2. 线性的向DataNode获取block块最终形成一个文件
3. 在blcok副本中按照距离优先进行选取
4. NN每次只给一部分block信息

### hdfs 的文件权限

1. 与linux文件权限类似,r:read;w:write;x:execute。权限x对于文件忽略，对于文件夹表示是否允许访问其内容。
2. 如果Linux系统用户zhangsan使用hadoop命令创建一个文件，那么这个文件在HDFS中owner就是zhangsan。
3. HDFS的权限目的：阻止好人错错事，而不是阻止坏人做坏事。HDFS相信，你告诉我你是谁，我就认为你是谁。

### hdfs 的安全模式

- namenode启动的时候，首先将映像文件(fsimage)载入内存，并执行编辑日志(edits)中的各项操作。
- 一旦在内存中成功建立文件系统元数据的映射，则创建一个新的fsimage文件(这个操作不需要SecondaryNameNode)和一个空的编辑日志。
- 此刻namenode运行在安全模式。即namenode的文件系统对于客户端来说是只读的。(显示目录，显示文件内容等。写、删除、重命名都会失败)。
- 在此阶段Namenode收集各个datanode的报告，当数据块达到最小副本数以上时，会被认为是“安全”的， 在一定比例（可设置）的数据块被确定为“安全”后，再过若干时间，安全模式结束
- 当检测到副本数不足的数据块时，该块会被复制直到达到最小副本数，系统中数据块的位置并不是由namenode维护的，而是以块列表形式存储在datanode中。

## hdfs2.X

### 背景

- hadoop1.0 中hdfs和Mapreduce在扩展性、高可用方面存在问题：
  - HDFS存在的问题
    - NameNode单点故障，难以应用于在线场景
    - NameNode压力过大，且内存受限，影扩展性
  - MapReduce存在的问题
    - JobTracker访问压力大，影响系统扩展性
    - 难以支持除MapReduce之外的计算框架，比如Spark、Storm等

### hdoop1.x和hadoop2.x

#### 架构比较

![hadoop1.0和hadoop2.0](https://img-blog.csdnimg.cn/20181227110824263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

- hadoop2.x由HDFS、MapReduce和YARN三个分支构成：
  - hdfs:支持ha.和federation联邦机制，2.0只支持2个节点的ha,3.0实现了一主多备
  - MapReduce：可以运行在yarn上
  - yarn：资源管理系统

#### hdfs2.0的优点

1. 解决HDFS 1.0中单点故障和内存受限问题。
   1. 解决单点故障：HDFS HA：通过主备NameNode解决，如果主NameNode发生故障，则切换到备NameNode上；
   2. 解决内存受限问题:HDFS Federation(联邦),水平扩展，支持多个NameNode；每个NameNode分管一部分目录；所有NameNode共享所有DataNode存储资源。
2. 2.x仅是架构上发生了变化，使用方式不变
3. 对HDFS使用者透明
4. HDFS 1.x中的命令和API仍可以使用

#### ha模型介绍

![ha模型](https://img-blog.csdnimg.cn/20181227112826766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA1NDE1OQ==,size_16,color_FFFFFF,t_70&ynotemdtimestamp=1555466135137)

- 主备NameNode 解决单点故障（属性，位置）
  - 主NameNode对外提供服务，备NameNode同步主NameNode元数据，以待切换，所有DataNode同时向两个NameNode汇报数据块信息（位置）。
  - JNN:集群（属性）
  - standby：备，完成了edits.log文件的合并产生新的image，推送回ANN
- 两种切换选择 手动切换：通过命令实现主备之间的切换，可以用HDFS升级等场合 自动切换：基于Zookeeper实现
- 基于Zookeeper自动切换方案 ZooKeeper Failover Controller：监控NameNode健康状态， 并向Zookeeper注册NameNode NameNode挂掉后，ZKFC为NameNode竞争锁，获得ZKFC 锁的NameNode变为active

#### Federation

- 通过多个namenode/namespace把元数据的存储和管理分散到多个节点中，使到namenode/namespace可以通过增加机器来进行水平扩展。
- 能把单个namenode的负载分散到多个节点中，在HDFS数据规模较大的时候不会也降低HDFS的性能。可以通过多个namespace来隔离不同类型的应用，把不同类型应用的HDFS元数据的存储和管理分派到不同的namenode中。
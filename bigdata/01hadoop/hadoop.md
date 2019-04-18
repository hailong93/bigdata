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
- 第二个副本：放置在第一个副本不同的 机架的节点上。
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
    - NameNode压力过大，且内存受限，影响扩展性
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

# mapreduce

## MR的原语

- 输入(k,v)格式的数据集，map映射成一个中间数据集(k,v)；
- "相同"的key为一组，调用一次reduce方法，方法内迭代这一组数据进行计算;

## configuration

- 单机

```xml
vim mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

vim yarn-sist.xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

- 集群ha

```xml
vim yarn-sist.xml
 <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
<property>
   <name>yarn.resourcemanager.ha.enabled</name>
   <value>true</value>
 </property>
 <property>
   <name>yarn.resourcemanager.cluster-id</name>
   <value>cluster1</value>
 </property>
 <property>
   <name>yarn.resourcemanager.ha.rm-ids</name>
   <value>rm1,rm2</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm1</name>
   <value>shizhan01</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm2</name>
   <value>shizhan05</value>
 </property>
 <property>
   <name>yarn.resourcemanager.zk-address</name>
   <value>shizhan05:2181shizhan06:2181,shizhan07:2181</value>
 </property>
```

- 验证 <http://192.168.8.111:8088>

## Map与reduce

- Map:
  - 读懂数据
  - 映射为kv模型
  - 并行分布式
  - 计算向数据移动
- Reduce
  - 数据全量/分量加工
  - Reduce中可以包含不同的key
  - 相同的key汇聚到一个reduce中
  - 相同的key调用一次reduce方法
- KV使用自定义数据类型
  - 作为参数传递，节省开发成本，提高程序自由度
  - Writable序列化，使其能够分布式程序数据进行交互
  - comparable比较器，实现具体排序

## 每个名词解释及关系

- 客户端进行切片**splits**
- 按照切片的数量启动相应数量的**maptask**
- 将**maptask**跑出的数据进行分区**partition**，排序存储到文件中；
- 经过**shuffler**,进行分组排序，调用设定好数量的**reducetask**
- 将结果输出到输出文件中

## mapreduce程序

### 运行测试例子

```shell
cd $HADOOP_HOME/share/hadoop/mapreduce
hadoop jar hadoop-mapreduce-examples-2.6.5.jar wordcount  /input  /output
```

### 自定义wordcount

- 代码如下

```java
public  class MyWC {
	public static class MyMapper extends Mapper<Object, Text, Text, IntWritable>{
		private static final IntWritable one=new IntWritable(1);
		private Text word=new Text();
		@Override
		protected void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
				throws IOException, InterruptedException {
			//默认切割符是" \t\n\r\f" 空格、换行、分页
			StringTokenizer str = new StringTokenizer(value.toString());
			while(str.hasMoreTokens()) {
				word.set(str.nextToken());
				context.write(word, one);
			}
		}
		
	}
	
	public static class MyReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
		private IntWritable result=new IntWritable();
		@Override
		protected void reduce(Text value, Iterable<IntWritable> nums,
				Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
			int count=0;
			for (IntWritable num : nums) {
				count+=num.get();
			}
			result.set(count);
			context.write(value, result);
		}
		
	}
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration(true);
		String[] inputArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
		if(inputArgs.length<2) {
			System.err.println("Usage: MyWC <in> [<in>...] <out>");
			System.exit(2);
		}
		Job job = Job.getInstance();
		
		job.setJarByClass(MyWC.class);
		job.setJobName("mywc");
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		job.setMapperClass(MyMapper.class);
		job.setReducerClass(MyReducer.class);
		
		job.setNumReduceTasks(1);
		//添加输入路径
		for(int i=0;i<inputArgs.length-1;i++) {
			Path inputFile = new Path(inputArgs[i]);
			FileInputFormat.addInputPath(job, inputFile);
		}
		//添加输出路径
		Path out = new Path(inputArgs[inputArgs.length-1]);
		if(out.getFileSystem(conf).exists(out)) {
			out = new Path(inputArgs[inputArgs.length-1]+System.currentTimeMillis());
		}
		FileOutputFormat.setOutputPath(job,out);
		
		System.exit(job.waitForCompletion(true)?0:1);
	}
}
```

- 打包
- 运行

```
hadoop jar mywc.jar com.sxt.study.mapreduce.MyWC /bigdata/train.txt /bigdata/out
```

## 源码分析

### 客户端源码

![1555575227103](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1555575227103.png)

- computSplitSize
  - 计算切片的大小，默认大小：Math.max(minSize, Math.min(maxSize, blockSize));其实就是blocksize;
- getBlockIndex
  - 返回切片起始偏移量所在的block索引
- makeSplit
  - split的信息包括：new FileSplit(file, start, length, hosts, inMemoryHosts);

### maptask的输入

- maptask.run(final JobConf job, final TaskUmbilicalProtocol umbilical)
  - 执行runNewMapper(job, splitMetaInfo, umbilical, reporter);
    - 创建一个mapper:ReflectionUtils.newInstance(taskContext.getMapperClass(), job);
    - 创建一个input format：ReflectionUtils.newInstance(taskContext.getInputFormatClass(), job);默认是TextInputFormat
    - 创建一个切片
    - 创建recordReader,input =new NewTrackingRecordReader<INKEY,INVALUE> (split, inputFormat, reporter, taskContext);其实创建了一个LineRecordReader对象
    - 初始化recordReader:input.initialize(split, mapperContext);
      - real.initialize(split, context);即LineRecordReader的初始化
        - 开启一个hdfs的输入流进行读取数据
        - 除去第一切片，从第二个切片开始起始偏移量从第二行开始，保证数据的完整性；
    - 调用mapper的run方法
      - nextKeyValue():给key,value赋值，并返回一个blooean类型的变量
      - getCurrentKey()：获取key的值
      - getCurrentValue()：获取value的值

### maptask的输出

- maptask.run(...)
  - runNewNapper(...)
    - output = new NewOutputCollector(...)
      - 反射：partitions
      - collector = createSortingCollector(job, reporter);
        - 反射collector(MapOutputBuffer)
        - collector.init(context);
          - 缓冲区:kvbuffer = new byte[maxMemUsage];默认100M
          - sorter:QuickSort.class
          - comparator
          - combiner,系统没有提供默认的combiner,如果客户实现了combiner,当在进行排序的时候会调用一次combiner,当溢出的小文件大于默认值3时也会触发程序调用一次combiner,所以当实现了combiner时系统会触发combiner一次或者多次。
          - SpillThread：数据向磁盘文件写出用的线程

### reducetask

- 拉取数据
- 进行归并排序
  - 分组比较器，取值的优先顺序：用户自定义的分组比较器-->用户定义的排序比较器-->默认的key比较器
- 会用一个迭代器表示拉取回来的数据。(真迭代器)它的nextkeyvalue()方法中判断了下一条数据和当前数据是否相等。
- reduce方法中的values也是一个迭代器(假迭代器),回调用nextkeyvalue方法，更新nextkeyissame
- 总结：reduce在取一组数据的时候内存中只会有两条数据，当前一条和下一条用于比较是否属于同一组，避免了内存溢出。

## 案例

### 案例一

- 需求：找出每个月气温最高的2天

- 数据样例：

  - 1949-10-01 14:21:02 34c
  - 1949-10-01 19:21:02 38c
  - 1949-10-02 14:21:02 36c
  - 1950-10-01 12:21:02 27c
  - ...

- 总结：

  - 知识点1：自定义分区

  ```
  public class WeatherPartitioner extends Partitioner<Weather, IntWritable>{
  
  	@Override
  	public int getPartition(Weather key, IntWritable value, int numPartitions) {
  		// TODO Auto-generated method stub
  		return key.getYear()%numPartitions;
  	}
  
  }
  ```

  - 知识点2：自定义排序器

  ```
  public class WeatherSortComparator extends WritableComparator{
  
  
  	public WeatherSortComparator() {
  		super(Weather.class, true);
  	}
  
  	@Override
  	public int compare(WritableComparable a, WritableComparable b) {
  		Weather w1=(Weather)a;
  		Weather w2=(Weather)b;
  		int y = w1.getYear()-w2.getYear();
  		if(y==0) {
  			int m = w1.getMonth()-w2.getMonth();
  			if(m==0) {
  				return -(w1.getTemperature()-w2.getTemperature());
  			}
  			return m;
  		}
  		return y;
  	}
  
  
  }
  ```

  - 知识点三：自定义分组排序器

  ```
  public class WeatherGroupingComparator extends WritableComparator{
  	public WeatherGroupingComparator() {
  		super(Weather.class,true);
  	}
  
  	@Override
  	public int compare(WritableComparable a, WritableComparable b) {
  		Weather w1=(Weather)a;
  		Weather w2=(Weather)b;
  		int y = w1.getYear()-w2.getYear();
  		if(y==0) {
  			return  w1.getMonth()-w2.getMonth();
  		}
  		return y;
  	}
  
  }
  ```

### 案例二

- 需求：推荐好友
- 数据样例：
  - tom hello hadoop cat
  - world hadoop hello hive
  - cat tom hive
  - mr hive hello
  - hive cat hadoop world hello mr
  - hadoop tom hive world
  - hello tom world hive mr
- 分析
  - 推荐的好友应该是间接好友最多的
  - 找一个标志位，标识是间接好友还是直接好友

### 案例三 计算pageRank

- 总结

  - 知识点1：mapreduce程序在windows下运行配置

  ```
  conf.set("mapreduce.app-submission.corss-paltform", "true");
  //如果分布式运行,必须打jar包
  //且,client在集群外非hadoop jar，这种方式启动,client中必须配置jar的位置
  conf.set("mapreduce.framework.name", "local");
  //这个配置,只属于,切换分布式到本地单进程模拟运行的配置
  //这种方式不是分布式,所以不用打jar包
  ```

  - 知识点2：通过配置设置全局变量

  ```
  //设置变量
  conf.setInt("runCount", i);
  //取出变量
  int runCount = context.getConfiguration().getInt("runCount", 1);
  ```

  - 知识点3：计数器

  ```
  //设置计数器
  context.getCounter(Mycounter.my).increment(j);
  //使用计数器
  long sum = job.getCounters().findCounter(Mycounter.my).getValue();
  ```

### 案例四：协同过滤推荐

- 名词解释：
  - ItemCf:基于item的协同过滤，通过用户对不同item的评分来评测item之间的相似性，基于item之间的相似性做出推荐。简单来讲就是：给用户推荐和他之前喜欢的物品相似的物品。
  - Co-occurrence Matrix(同现矩阵):每个商品与其他商品一起出现的次数。以每个用户为维度。
  - User Preference Vector(用户评分向量):用户对一些商品的评分，比如通过点击、加入购物车、购买等不同的操作可以体现出不同的分数。
  - Recommended Vector(推荐向量)：同现矩阵*用户评分向量矩阵
- 实现
  1. 去除重复数据
  2. 计算用户评分向量矩阵
  3. 计算同现矩阵
  4. 同现矩阵与用户评分矩阵相乘
  5. 矩阵求和
  6. 取topN

# yarn

## 背景

* 多集群运维成本高
* 跨集群移动数据成本高，资源浪费
* 离线计算，实时计算，算法模型训练等不同类型计算需要优化计算资源

## 简介

Hadoop2.0后开始引入YARN

功能：

* 集群资源管理系统
* 负责集群的同一管理和调度
* 与客户端交互，处理客户端请求

## 基本架构

![1555505529979](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1555505529979.png)

* **ResourceManager**
  * 处理客户请求
  * 启动和监控ApplicationMaster
  * 监控NodeManager
  * 负责整个集群的资源管理和调度
* **NodeManager**
  * 每个节点只有一个，一般与DataNode一一对应，在相同的机器上部署
  * 负责节点资源的监控和管理，定时向ResourceManager汇报信息。
  * 处理来自ResourceManager的请求，为作业的执行分配Container
  * 处理来自ApplicationMaster的请求，启动和停止Container.

* **ApplicationMaster**
  * 每个应用程序只有一个，负责应用程序的管理，资源申请和任务调度。
  * 与ResourceManager协商为应用程序申请资源
  * 与NodeManager通信启动或停止任务
  * 监控任务运行状态和失败处理。

* **Container**
  * 任务运行环境的抽象，只有在分配任务的时候才会抽象出一个Contatiner
  * 任务运行资源(节点、内存、CPU)
  * 任务启动命令
  * 任务运行环境

* 流程：
  * 客户端发起一个任务的计算的请求到resourceManager;
  * resourcemanager跟nodeManager进行通信，让其启动一个基于cpu和内存的container容器;
  * 针对这个计算程序ResourceManager会启动一个ApplicationMaster,负责具体任务的资源调度和监控;
  * applicationmaster会和一个nodemanager进行通信，启动一个container执行具体的任务;
  * 这个任务会实时的和application汇报执行的状态，直至任务运行结束。

## yarn容错性

1. ResourceManager
   * 基于Zookeeper实现高可用
2. NodeManager
   * NodeManager故障导致运行在该节点的任务失败，任务失败后ResourceManger将失败任务通知对应的ApplicationMaster.
   * ApplicationMaster决定如何处理失败的任务。
3. ApplicationMaster
   * ApplicationMaster失败后，由ResourceManager负责重启。

# hadoop安装

## 前置条件

1. 安装jdk

   - jps

2. 关闭防火墙

   ```shell
   service iptables status
   service iptables stop
   chkconfig iptables off
   ```

3. ssh免密dsa模式

   ```shell
   ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
   cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
   ```

## 单机/伪分布式

1. 上传hadoop安装包并解压

   ```shell
   tar xf hadoop.tar.gz
   mv hadoop-2.6.5 /opt/bigdata
   ```

2. 配置环境变量

   ```shell
   vim /etc/profile
   export JAVA_HOME=/usr/java/jdk1.7.0_67
   export HADOOP_PREFIX=/opt/bigdata/hadoop-2.6.5
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_PREFIX/bin:$HADOOP_PREFIX/sbin
   ```

3. 配置配置文件

   - [hadoop-env.sh](http://hadoop-env.sh) [mapred-env.sh](http://mapred-env.sh) [yarn-env.sh](http://yarn-env.sh) 添加java_home的地址
   - vim core-site.xml

   ```xml
   <configuration>
    <property>
         <name>fs.defaultFS</name>
           <value>hdfs://shizhan01:9000</value>
       </property>
   
    <property>
           <name>hadoop.tmp.dir</name>
           <value>/var/bigdata/hadoop/alone</value>
    </property>
   </configuration>
   ```

   - vim hdfs-site.xml

   ```xml
   <configuration>
       <property>
               <name>dfs.replication</name>
               <value>1</value>
       </property>
       
       <property>
               <name>dfs.namenode.secondary.http-address</name>
               <value>shizhan01:50090</value>
       </property>
   </configuration>
   ```

   - vim slaves

   ```sehll
   shizhan01
   ```

4. 格式化namenode

   ```shell
    hdfs namenode -format
   ```

5. 启动

   ```shell
   start-dfs.sh
   ```

6. 网页地址：<http://192.168.9.11:50070>

## 分布式

1. 免密

   ```shell
   sch id_dsa.pub shizhan05:`pwd`/shizhan01_dsa.pub
   sch id_dsa.pub shizhan06:`pwd`/shizhan01_dsa.pub
   sch id_dsa.pub shizhan07:`pwd`/shizhan01_dsa.pub
   cat ~/.ssh/shizhan01_dsa.pub >> ~/.ssh/authorized_keys
   ```

2. 上传hadoop安装包并解压

   ```shell
   tar xf hadoop.tar.gz
   mv hadoop-2.6.5 /opt/bigdata
   ```

3. 配置环境变量

   ```shell
   vim /etc/profile
   export JAVA_HOME=/usr/java/jdk1.7.0_67
   export HADOOP_PREFIX=/opt/bigdata/hadoop-2.6.5
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_PREFIX/bin:$HADOOP_PREFIX/sbin
   ```

4. 配置配置文件

   - [hadoop-env.sh](http://hadoop-env.sh) [mapred-env.sh](http://mapred-env.sh) [yarn-env.sh](http://yarn-env.sh) 添加java_home的地址
   - vim core-site.xml

   ```xml
   <configuration>
    <property>
           <name>fs.defaultFS</name>
           <value>hdfs://shizhan01:9000</value>
       </property>
   
    <property>
           <name>hadoop.tmp.dir</name>
           <value>/var/bigdata/hadoop/cluster</value>
       </property>
   </configuration>
   ```

   - vim hdfs-site.xml

   ```xml
   <configuration>
       <property>
               <name>dfs.replication</name>
               <value>2</value>
           </property>
       
       
       <property>
               <name>dfs.namenode.secondary.http-address</name>
               <value>shizhan06:50090</value>
       </property>
   </configuration>
   ```

   - vim slaves

   ```shell
   shizhan06
   shizhan07
   ```

5. 分发部署包到其他节点

   ```shell
   cd /opt/sxt
   scp -r hadoop-2.6.5  shizhan06:`pwd`
   scp -r hadoop-2.6.5  shizhan07:`pwd`
   ```

6. 启动

   ```shell
   start-dfs.sh
   ```

7. 网页地址：<http://192.168.9.11:50070>

## ha集群搭建

1. 安装zookeeper

   - 配置环境变量

   ```shell
   export ZOOKEEPER_HOME=/opt/bigdata/zookeeper-3.4.6
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin
   ```

   - 配置配置文件

   ```shell
   cp zoo_sample.cfg zoo.cfg
   vim zoo.cfg
   
   dataDir=/var/bigdata/zk
   server.1=shizhan05:2888:38888
   server.2=shizhan06:2888:38888
   server.3=shizhan07:2888:38888
   
   echo 1 > /var/bigdata/zk/myid
   ```

   - 分发到每个节点

   ```shell
   scp -r zookeeper-3.4.6 shizhan06:`pwd`
   scp -r zookeeper-3.4.6 shizhan07:`pwd`
   ```

   - 在shizhan06和shizhan07上分别创建myid并赋上相应的id号

   ```shell
   echo 2 > /var/bigdata/zk/myid
   echo 3 > /var/bigdata/zk/myid
   ```

   - 启动zookeeper

   ```shell
   zkServer.sh start
   ```

2. 配置文件

   - [hadoop-env.sh](http://hadoop-env.sh) [mapred-env.sh](http://mapred-env.sh) [yarn-env.sh](http://yarn-env.sh) 添加java_home的地址
   - vim core-site.xml

   ```xml
   <configuration>
   
    <property>
           <name>hadoop.tmp.dir</name>
           <value>/var/bigdata/hadoop/ha</value>
       </property>
   
    <property>
     <name>fs.defaultFS</name>
     <value>hdfs://mycluster</value>
   </property>
   
   <property>
      <name>ha.zookeeper.quorum</name>
      <value>shizhan05:2181,shizhan06:2181,shizhan07:2181</value>
    </property>
   
   </configuration>
   ```

   - vim hdfs-site.xml

   ```xml
   <configuration>
   
   <property>
           <name>dfs.replication</name>
           <value>2</value>
       </property>
       
     <property>
     <name>dfs.nameservices</name>
     <value>mycluster</value>
   </property>
   
   <property>
     <name>dfs.ha.namenodes.mycluster</name>
     <value>nn1,nn2</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.mycluster.nn1</name>
     <value>shizhan01:8020</value>
   </property>
   <property>
     <name>dfs.namenode.rpc-address.mycluster.nn2</name>
     <value>shizhan05:8020</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.mycluster.nn1</name>
     <value>shizhan01:50070</value>
   </property>
   <property>
     <name>dfs.namenode.http-address.mycluster.nn2</name>
     <value>shizhan05:50070</value>
   </property>
   
   <property>
     <name>dfs.namenode.shared.edits.dir</name>
     <value>qjournal://shizhan01:8485;shizhan05:8485;shizhan06:8485/mycluster</value>
   </property>
   
   <property>
     <name>dfs.journalnode.edits.dir</name>
     <value>/var/bigdata/hadoop/ha/jn</value>
   </property>
   
   <property>
     <name>dfs.client.failover.proxy.provider.mycluster</name>
     <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
   </property>
   <property>
     <name>dfs.ha.fencing.methods</name>
     <value>sshfence</value>
   </property>
   <property>
     <name>dfs.ha.fencing.ssh.private-key-files</name>
     <value>/root/.ssh/id_rsa</value>
   </property>
   
   <property>
      <name>dfs.ha.automatic-failover.enabled</name>
      <value>true</value>
    </property>
   
   
   
   </configuration>
   ```

3. 启动journalnode集群

   ```shell
   hadoop-daemon.sh start journalnode
   ```

4. 第一台namenode

   ```shell
   hdfs namenode -format
   hadoop-daemon.sh start namenode
   ```

5. 第二台namenode

   ```shell
   hdfs namenode -bootstrapStandby
   ```

6. 格式化zkfc在第一台namenode机器上执行就好

   ```shell
   hdfs zkfc -formatZK
   ```

7. 启动hdfs

   ```shell
   start-dfs.sh
   ```
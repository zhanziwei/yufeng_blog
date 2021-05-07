---
layout: post
title: mapreduce概述
date: 2021-04-19
Author: yufeng 
tags: [Hadoop]
comments: true
toc: true
---

## MapReduce详解

#### MapReduce是什么

MapReduce是一种分布式的计算框架，一个MapReduce作业把输入的数据集切分为若干独立的数据块，由Map任务以完全并行的方式去处理。框架对Map的输出进行排序后把结果输入给Reduce任务，Reduce对map阶段的结果进行汇总。

#### MapReduce的优缺点

##### 优点

1. 易于编程
2. 良好的扩展性
3. 高容错性
4. 适合海量数据的离线处理

##### 缺点

1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长有向无环图计算

#### Wordcount编程实例

1. Mapper

   ```java
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.LongWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Mapper;
   
   import java.io.IOException;
   
   // 1. 继承Mapper类
   public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
       
       Text k = new Text();
       IntWritable v = new IntWritable(1);
   	// 2. 重写Map方法 
       @Override
       protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
           // 3. 得到每行数据
           String line = value.toString();
   
           String[] words = line.trim().split("\t");
   
           for(String word:words) {
               // 4. 将单词写入key中
               k.set(word);
               // 5. 输出键值对，v初始化为1
               context.write(k, v);
           }
   
   
       }
   }
   
   ```

2. Reducer

   ```java
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.Reducer;
   
   import java.io.IOException;
   
   //1. 继承Reducer类
   public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
       int sum;
       IntWritable v = new IntWritable();
   	// 2. 重写reduce方法
       @Override
       protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
           sum = 0;
   		// 对于输入的键值对为（word, list of count），对每个单词进行累加
           for(IntWritable count:values) {
               sum += count.get();
           }
   
           v.set(sum);
           // 输出键值对为（word, sum）
           context.write(key, v);
       }
   }
   ```

3. Driver

   Driver阶段相当于Yarn集群的客户端，用于提交我们整个程序到Yarn集群，提交的是封装了MapReduce程序相关运行参数的job对象

   ```java
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IntWritable;
   import org.apache.hadoop.io.Text;
   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
   import org.apache.hadoop.mapreduce.Job;
   
   import java.io.IOException;
   
   public class WordCountDriver {
       public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
           // 1. 获取默认的配置文件信息
           Configuration conf = new Configuration();
           // 2. 通过配置信息初始化job实例
           Job job = Job.getInstance(conf);
           
           // 3. 通过class名称得到driver的jar包
           job.setJarByClass(WordCountDriver.class);
   		// 4. 设置mapper和reducer的实例
           job.setMapperClass(WordCountMapper.class);
           job.setReducerClass(WordCountReducer.class);
   		// 5. 设置Map的输出数据的类型
           job.setMapOutputKeyClass(Text.class);
           job.setMapOutputValueClass(IntWritable.class);
   		
           // 6. 设置作业输出数据的类型
           job.setOutputKeyClass(Text.class);
           job.setOutputValueClass(IntWritable.class);
   
   		// 7. 设置输入路径和输出入境
           FileInputFormat.setInputPaths(job, new Path(args[0]));
           FileOutputFormat.setOutputPath(job, new Path(args[1]));
   		// 8. 通过waitForCompletion每秒轮询作业的进度，当发现自上次报告有改变，便把进度报告到控制台
           boolean result = job.waitForCompletion(true);
           System.exit(result?0:1);
       }
   }
   ```

4. 执行WordCount程序

   ``hadoop jar wordcount.jar com.zhanziwei.WordCountDriver /input路径 /output路径``

#### MapReduce工作流

1. 使用JobControl去控制工作流的进行

   JobControl类的实例表示一个作业的运行图，可以加入作业配置，然后告知JObControl实例作业之间的依赖关系，在一个线程中运行JobControl时，将按照依赖顺序来执行这些作业。

2. 使用Apache Oozie

   Oozie是一个运行工作流的系统，由两部分组成：一个工作流引擎，负责存储和运行由不同类型的Hadoop作业组成的工作流；一个coordinator引擎，负责基于预定义的调度策略及数据可用性运行工作流作业。

   Oozie的优点：考虑到了可扩展性，容易处理失败工作流的重运行。

   Oozie作为服务器运行，客户端提交一个立即或稍后执行的工作流定义到服务器。在Oozie中，工作流由动作节点和控制流节点组成的DAG（有向无环图）。

   动作节点执行工作流任务，控制流节点通过构建条件逻辑或并行执行来管理活动之间的工作流执行情况。当工作流结束后，Oozie通过发送一个HTTP的回调向客户端通知工作流的状态。还可以在每次进入工作流或退出一个动作节点时接收到回调。

#### Oozie实例

1. 定义Oozie工作流
2. 打包和配置Oozie工作流应用
3. 运行Oozie工作流作业

#### MapReduce的运行机制

<img src="https://pic2.zhimg.com/80/v2-35602b384f731eeca77c1f4f5e2e46bd_720w.jpg" alt="img" style="zoom:80%;" />

1. 作业的提交

   Job的submit方法创建一个内部的JobSubmitter实例，调用其submitJobInternal方法，提交作业后，waitForCompletion每秒轮询作业的进度，如果发现有所改变，则将进度报告到控制台。完成后，如果成功，显示作业计数器；失败则打印失败错误。

   作业的提交过程包括：

   * 向ResourceManager请求一个新应用ID，用于MapReduce作业ID
   * 检查作业的输出说明
   * 计算作业的输入分片
   * 将运行作业所需要的资源复制到一个以作业ID命名的目录下的共享文件系统中
   * 调用ResourceManager的submitApplication方法提交作业

2. 作业的初始化

   ResourceManager收到调用它的submitApplication消息后，将请求传递给Yarn调度器，调度器分配一个容器，然后资源管理器在节点管理器的管理下在容器中启动application master进程。MRAPPMaster接收来自任务的进度和完成报告；接下来接收来自共享文件系统，在客户端计算的输入分片，对每一个分片创建一个map任务对象以及确认多个reduce任务，分配任务ID，当作业较小时，发生uber任务。小作业为少于10个mapper且只有1个reducer且输入大小小于一个HDFS块的作业。

   最后在任何任务运行之前，application master调用setupJob方法设置outputCommiter，表示将建立作业的最终输出目录以及任务输出的临时工作空间。

3. 任务的分配

   如果不作为uber任务，则application master为该作业中所有map任务和reduce任务向资源管理器请求容器，Map的请求优先级高于reduce，直到有5%的map任务已经完成时，才会为reduce任务的请求发出。

4. 任务的执行

   当分配了一个特定节点上的容器，application master通过与节点管理器通信启动容器，该任务由主类为YarnChild的Java应用程序执行。在运行任务前，先将资源本地化，最后运行任务。

   YarnChild 在指定的JVM中运行，map或reduce中的缺陷不会影响到节点管理器

   Streaming运行特殊的map任务和reduce任务，目的是运行用户提供的可执行程序并与之通信。

5. 进度和状态的更新

   任务在运行时，对其进度保持追踪；对于map任务，任务进度是已处理输入所占的比例。对reduce任务，整个过程分为三个部分：复制、排序和reduce。

   当任务运行时，子进程和自己的父application  master通过umbilical接口通信，每隔3s报告一次。

   客户端每秒钟轮询一个application master接收最新状态

6. 作业的完成

   当application master收到作业最后一个任务已完成的通知，则把作业的状态设置为成功。然后再Job轮询状态时，直到任务已经成功完成，打印消息给用户并从waitForCompletion方法返回。

   最后作业完成时，application master和任务容器清理其工作状态，OutputCommiter的commitJob方法被调用。作业信息由作业历史服务器存档，便于日后查询。

#### MapReduce作业失败

1. 任务运行失败

   当Applicatin master注意到由有一段时间没收到进度的更新，则把任务标记为失败。将重新调度该任务的执行

2. application master运行失败

   当Application master失败时，资源管理器将检测到该失败并在一个新的容器中开始一个新的master实例。

   在作业初始化期间，客户端向资源管理器询问并缓存applicatin master的地址，当失败后，客户端会向资源管理器请求新的application master地址

3. 节点管理器运行失败

   资源管理器从节点池中移除该节点、

4. 资源管理器运行失败

   使用高可用资源管理器。把所有运行中的应用程序信息存储在一个高可用的状态存储区中，备机可以恢复出失败的主机关键状态。节点管理器发送第一个心跳信息时，节点管理器信息能以相当快的速度被新的资源管理器重构。

   当新资源管理器启动后，从状态存储区中读取应用程序的信息，为集群中运行的所有应用程序重启application master，不被计为失败的应用程序。

   资源管理器从备机切换到主机是由故障转移控制器处理的。使用Zookeeper的leader选举机制确保同一时刻只有一个主机。

#### InputFormat数据输入

在MapReduce中，数据块是HDFS的存储数据单元，数据切片是逻辑上对输入进行分片，每个切片对应一个map任务。

#### Job提交流程

```java
1. 执行job，等待完成
boolean result = job.waitForCompletion(true);

2. 执行submit方法
public boolean waitForCompletion(boolean verbose) throws IOException, InterruptedException, ClassNotFoundException {
        if (this.state == Job.JobState.DEFINE) {
            this.submit();
        }
}

public void submit() throws IOException, InterruptedException, ClassNotFoundException {
        this.ensureState(Job.JobState.DEFINE);
        this.setUseNewAPI();
    	2.1 建立连接
        this.connect();
    
}


private synchronized void connect() throws IOException, InterruptedException, ClassNotFoundException {
        if (this.cluster == null) {
            this.cluster = (Cluster)this.ugi.doAs(new PrivilegedExceptionAction<Cluster>() {
                public Cluster run() throws IOException, InterruptedException, ClassNotFoundException {
                    2.2 创建提交Job的代理
                    return new Cluster(Job.this.getConfiguration());
                }
            });
        }
}

判断是local还是Yarn运行环境
if (jobTrackAddr == null) {
                    clientProtocol = provider.create(conf);
                } else {
  
                    clientProtocol = provider.create(jobTrackAddr, conf);
                }

创建给集群提交数据的Staging路径
Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf); 

获取jobid并创建Job路径
JobID jobId = submitClient.getNewJobID(); 

拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);

计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
maps = writeNewSplits(job, jobSubmitDir);
input.getSplits(job);

向Stag路径写XML配置文件，写入Job相关参数
writeConf(conf, submitJobFile);
conf.writeXml(out);

提交Job，返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(),job.getCredentials());
```

![img](https://upload-images.jianshu.io/upload_images/24133934-8a7de7a2cd379b8a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### FileInputFormat切片解析

input.getSplits(job)

1. 程序找到数据存储的目录
2. 遍历处理目录下的每一个文件
3. 遍历第一个文件
   * 获取文件大小fs.sizeOf(fileName)
   * 计算切片大小computeSplitSize(Math.max(minSize,Math.min(maxSize,blockSize)))
   * 开始切，形成切片，每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片
   * 将切片信息写到一个切片规划文件中
   * 整个切片的核心过程在getSplit方法中完成
   * InputSplit只记录切片的元数据信息，如起始位置、长度以及所在的节点列表等
4. 提交切片规划文件到Yarn上，Yarn的MRAPPMaster根据切片规划文件计算开始MapTask个数

切片由InputFormat抽象类创建，通过input.getSplits(Job)计算分片，返回所有分片信息的集合。Map任务把输入切片传给InputFormat中的createRecordReader()获取RecordReader，转换为键值对传给map函数。

#### 避免切分

一个map去处理一整个文件

1：将最小分块大小设置为大于文件大小

2：使用FileInputFormat具体子类，重写isSplitable方法，返回值设置为false。

#### 将这个文件作为一条记录

写一个WholeFileInputFormat类继承FileInputFormat，重写isSplitable和createRecordReader方法，不切分文件，把文件内容放进一个字节数组。

#### 小文件的弊端

1. 增加map开销
2. 处理小文件，增加作业的寻址速度
3. 需要记录小文件的元数据，造成namenode内存浪费

#### 小文件的解决办法

1. 将多个小文件事先合成一个顺序文件：将文件名为key，文件内容为值

2. 使用CombineFileInputFormat将多个小文件打包到一个分片中

   切片机制包括虚拟存储过程和切片过程两部分。

   CombineTextInputFormat.setMaxInputSplitSize(job, 4194304)

   * 虚拟存储过程

     将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不 大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍， 那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时 将文件均分成 2 个虚拟存储块（防止出现太小切片）。

   * 切片过程

     * 判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片
     * 如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片

#### 不同的输入格式

1. TextInputFormat

   默认的FileInputFormat实现类，按行读取每条记录，键是存储该行在整个文件中的起始字节偏移量，值为这行的内容

2. KeyValueTextInputFormat

   文件中的每一行是一个键值对，使用某个分界符进行分隔

3. NLineInputFormat

   使得mapper得到固定行数的输入，键是文件中行的字节偏移量，值为行本身

4. SequenceFileInputFormat

   顺序文件格式，存储二进制的键值对的序列，键和值由顺序文件决定，要保证map输入的类型匹配。

   SequenceFile文件包括文件头、文件记录、同步标识。

#### MapReduce的计算过程

![preview](https://pic2.zhimg.com/v2-bd1d4a527fba9a9d265cd05e990ed455_r.jpg)

##### Map端

1. **Read阶段**：由程序内的InputFormat（默认实现类为TextInputFormat）来读取外部数据，它调用RecordReader（它的成员变量）的read方法来读取，返回k,v键值对

2. **Map阶段**：读取的k，v键值对传送给map方法，作为其入参来执行用户定义的map逻辑。

3. **Collect阶段**：context.write方法被调用时，outputCollector.collect方法将map方法的输出结果写入到环形缓冲区中。

4. **Spill阶段**：环形缓冲区就是一个数组，后端不断接受数据的同时，前端数据不断被溢出，长度用完后读取的新数据再从前端开始覆盖。默认大小为100M，通过MR.SORT.MB配置，当缓冲的容量达到默认大小的80%时，进行反向溢写。

   Spill详情：

   * 默认使用快排队缓冲区内的数据排序，先按照分区编号Partition，再按照key进行排序，形成分区并分区内所有数据按key有序
   * 按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件，如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行聚集
   * 将分区数据的元信息写到内存索引数据结构SpillRecord中，每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。
   * Combine阶段：临时文件执行merge，以分区为单位进行合并，会再一次调用Combiner，合并成一个大文件

每个MapTask最终只生成一个数据文件，避免同时打开大量文件和同时读取大量小文件产生的IO开销。

##### Reduce端

![img](https://pic2.zhimg.com/80/v2-b26eeb521fd5e03f5c835c7c237bbef1_720w.jpg)

1. **Copy阶段**：Reduce task从各个MapTask上远程拷贝一片数据，如果大小超过阈值，则写到磁盘，否则放到内存
2. **Merge阶段**：远程拷贝数据的同时，ReduceTask启动两个后台线程对内存和磁盘上的文件进行合并，防止内存使用过多或磁盘上文件过多
3. **Sort阶段**：通过GroupingComparator()分辨同一组数据，把它们发送给reduce(k, iterator)方法
4. **Reduce阶段**：调用context.write()方法，让OutputFormat调用RecordWriter的write()方法将处理结果写入到数据仓库中，写出的只有一个分区的文件数据。

#### Shuffle机制

**map端**：每个Map任务维护着一个环形缓冲区，当map输出数据时，先写到缓冲区内，当缓冲区内容达到设置的阈值之后，溢出成spill文件，通过Partitioner进行分区，针对key排序，有combiner，则排序后运行combiner，使结果更紧凑，写入磁盘，当最后一个spill文件写完之后，将多个spill文件进行合并到一个已经分区并排序的大文件，如果有combiner，在合并spill时，也会运行。压缩的Map输出可以减少磁盘的IO量，减少传输给reduce的数据量。

**reduce端**：Reduce通过HTTP方式从map获取数据，Reduce有少量的复制线程，并行从map端复制数据到Reduce端。一般需要从多个map端复制数据，有一个map完成就可以开始复制。若map输出比较小，则复制到内存；如果数据大，当达到内存缓冲区的阈值，会合并溢出到磁盘。如果有combiner，合并期间运行，降低写入磁盘的数据量。对已经排序输出的每一个键调用reduce函数，然后输出到HDFS。

##### shuffle缺陷

1. 每个map可能会有多个spill文件需要写入磁盘，产生较多的磁盘IO
2. 数据量很小，但map和reduce任务很多时，产生较多的网络IO

### Combiner函数何时被使用

1. Map端溢出spill文件后，进行分区排序后，会执行combiner函数
2. 多个spill文件合并成一个大文件时，会使用
3. 在Reduce任务端，复制map输出，在合并后溢出到磁盘时也会执行combiner函数

#### 为什么要使用combiner

减少本地磁盘io；减少reduce端复制数据的网络io；将上述优化与业务逻辑剥离，使得作业调优与业务逻辑之间的耦合度降低

#### 什么时候用Combiner

对于求最值或者求和之类的统计可以设置Combiner，只要不影响最终的reduce结果。

#### Partition分区

默认的partition为HashPartitiner，key.hashCode()&Integer.MAX_VALUE % numReduceTasks，用户不能控制key存储到哪个分区

自定义Partitioner

```java
//1. 自定义类继承Partitioner，重写getPartition方法
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
    @Override
    public int getPartition(Text text, FlowBean flowBean, int i) {
        // 控制分区代码逻辑
        return partition;
    }
}

//2. 在Job驱动中设置自定义Partitioner
job.setPartitionerClass(CustomPartitioner.class);

//3. 自定义后，根据自定义Partitioner的逻辑设置相应数量的ReduceTask
job.setNumReduceTasks(5);
```

#### 排序

1. 部分排序

   根据输入记录的键对数据集排序，保证输出的每个文件内部有序

2. 全排序

   最终输出结果只有一个文件，且文件内部有序

   1. 设置一个分区，使用一个Reduce进行排序，丧失了mapreduce的并行架构

   2. 自定义分区函数的分界点，每个分区之间有顺序，分区内局部有序，从而全局有序，容易发生数据倾斜

   3. 基于数据采样的全局排序

      * 对待排序数据进行抽样
      * 对抽样数据进行排序，产生分割点
      * Map对输入的每条数据计算其处于哪两个分割点之间；将数据发给对应区间ID的reduce
      * Reduce获得数据直接输出

      关键点：

      * 数据采样，保存key的分割点，采样器包括SplitSampler、IntervalSampler、RandomSampler，根据最大样本数、最大分区和输入分片确定每个分片的采样数。
      * 根据reduce设定数目，确定分区并根据采样设置分割点
      * 将key分割点存储在指定的hdfs系统
      * 使用TotalOrderPartitioner设置为分区函数

3. 辅助排序

   对key和value都进行排序

   1. 将自然键和自然值合并成组合键
   2. map输出后，按照组合键进行分区和排序，自定义分区函数和排序函数
   3. 针对组合键进行分区和分组时只考虑自然键

   ```java
   // IntPair为组合键
   public class FirstPartitioner extends Partitioner<IntPair,IntWritable> {
       @Override
       public int getPartition(IntPair key, IntWritable value, int numPartitions) {
           return Math.abs(key.getFirst() * 127) % numPartitions;
       }
   }
   // 根据key.first进行分区，泛型类型与mapper输出一致
   
   public class KeyComparator extends WritableComparator {
       protected KeyComparator() {
           super(IntPair.class,true);//需要实例化
       }
   
       @Override
       public int compare(WritableComparable a, WritableComparable b) {
           IntPair p1=(IntPair)a;
           IntPair p2=(IntPair)b;
           int result = IntPair.compare(p1.getFirst(),p2.getFirst());
           if(result==0){
               result = -IntPair.compare(p1.getSecond(),p2.getSecond()); //前面加一个减号求反
           }
           return result;
       }
   }
   // key比较器，用来map阶段的key排序使用,根据first升序，second降序排序
   
   public class GroupComparator extends WritableComparator {
       public GroupComparator() {
           super(IntPair.class,true);
       }
   
   
       @Override
       public int compare(WritableComparable a, WritableComparable b) {
           IntPair p1=(IntPair)a;
           IntPair p2=(IntPair)b;
           return IntPair.compare(p1.getFirst(),p2.getFirst());
       }
   }
   // 分组比较器，确保同一个年份的数据在同一个组里，只按年份进行比较
   ```

   分区是将有关联的键划分给同一个reducer处理，多一个分区，则多一个Reducer

   分组是把相同的键的记录整理为一组记录，在同一个分区里面，具有相同key值得记录是属于同一个分组。

#### 连接

##### Map端连接

1. 使用场景：适用于一张表小，一张表很大的场景

2. 优点：在Reduce端处理过多的表，容易产生数据倾斜，通过在Map端缓存多张数据表，提交处理业务逻辑，增加Map端业务，较少Reduce端数据的压力，减少数据倾斜的可能。

3. 具体方法：采用DistributedCache

   * 在Mapper的setup阶段，将文件读取到缓存集合中

     ```java
     public class DistributedCacheMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
     
     	Map<String, String> pdMap = new HashMap<>();
     	
     	@Override
     	protected void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context) throws IOException, InterruptedException {
     
     		// 1 获取缓存的文件
     		URI[] cacheFiles = context.getCacheFiles();
     		String path = cacheFiles[0].getPath().toString();
     		
     		BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(path), "UTF-8"));
     		
     		String line;
     		while(StringUtils.isNotEmpty(line = reader.readLine())){
     
     			// 2 切割
     			String[] fields = line.split("\t");
     			
     			// 3 缓存数据到集合
     			pdMap.put(fields[0], fields[1]);
     		}
     		
     		// 4 关流
     		reader.close();
     	}
     	
     	Text k = new Text();
     	
     	@Override
     	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
     
     		// 1 获取一行
     		String line = value.toString();
     		
     		// 2 截取
     		String[] fields = line.split("\t");
     		
     		// 3 获取产品id
     		String pId = fields[1];
     		
     		// 4 获取商品名称
     		String pdName = pdMap.get(pId);
     		
     		// 5 拼接
     		k.set(line + "\t"+ pdName);
     		
     		// 6 写出
     		context.write(k, NullWritable.get());
     	}
     }
     ```

     

   * 在驱动函数中加载缓存，缓存普通文件到Task运行节点

     ```java
     job.addCacheFile(new URI("file:///e:/小表文件"))
     
     job.setNumReduceTasks(0)  // Map端join的逻辑不需要Reduce阶段，设置reduceTask数量为0
     ```

##### reduce端连接

通过将关联条件作为Map输出的key，将两表满足Join条件的数据并携带数据所在源的文件信息，发往同一个ReduceTask，在Reduce中进行数据的串联

```java
// 1. 创建商品和订单合并后的Bean类
public class TableBean implements Writable {
    private String order_id;
    private String p_id;
    private int amount;
    private String pname;
    private String flag;
    
    @Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(order_id);
		out.writeUTF(p_id);
		out.writeInt(amount);
		out.writeUTF(pname);
		out.writeUTF(flag);
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.order_id = in.readUTF();
		this.p_id = in.readUTF();
		this.amount = in.readInt();
		this.pname = in.readUTF();
		this.flag = in.readUTF();
	}
	... getter and setter
}

// 2. 编写mapper类
public class TableMapper extends Mapper<LongWritable, Text, Text, TableBean> {
    String name;
    TableBean bean = new TableBean();
    Text k = new Text();
    
    @Override
	protected void setup(Context context) throws IOException, InterruptedException {

		// 1 获取输入文件切片
		FileSplit split = (FileSplit) context.getInputSplit();

		// 2 获取输入文件名称
		name = split.getPath().getName();
	}
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取输入数据
		String line = value.toString();
		
		// 2 不同文件分别处理
		if (name.startsWith("order")) {// 订单表处理

			// 2.1 切割
			String[] fields = line.split("\t");
			
			// 2.2 封装bean对象
			bean.setOrder_id(fields[0]);
			bean.setP_id(fields[1]);
			bean.setAmount(Integer.parseInt(fields[2]));
			bean.setPname("");
			bean.setFlag("order");
			
			k.set(fields[1]);
		}else {// 产品表处理

			// 2.3 切割
			String[] fields = line.split("\t");
			
			// 2.4 封装bean对象
			bean.setP_id(fields[0]);
			bean.setPname(fields[1]);
			bean.setFlag("pd");
			bean.setAmount(0);
			bean.setOrder_id("");
			
			k.set(fields[0]);
		}

		// 3 写出
		context.write(k, bean);
	}
}

// 3. 编写TableReducer类
public class TableReducer extends Reducer<Text, TableBean, TableBean, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<TableBean> values, Context context)	throws IOException, InterruptedException {

		// 1准备存储订单的集合
		ArrayList<TableBean> orderBeans = new ArrayList<>();
		
// 2 准备bean对象
		TableBean pdBean = new TableBean();

		for (TableBean bean : values) {

			if ("order".equals(bean.getFlag())) {// 订单表

				// 拷贝传递过来的每条订单数据到集合中
				TableBean orderBean = new TableBean();

				try {
					BeanUtils.copyProperties(orderBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}

				orderBeans.add(orderBean);
			} else {// 产品表

				try {
					// 拷贝传递过来的产品表到内存中
					BeanUtils.copyProperties(pdBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}

		// 3 表的拼接
		for(TableBean bean:orderBeans){

			bean.setPname (pdBean.getPname());
			
			// 4 数据写出去
			context.write(bean, NullWritable.get());
		}
	}
}
```

#### 分布式缓存

将指定文件复制到分布式系统，

``job.addCacheFile(new URI("/foo/bar/fileName.txt#fileName.txt"))``

工作机制：当用户启动一个作业，Hadoop会把由-files、-archives和-libjars等选项指定的文件复制到分布式文件系统。在任务运行之前，节点管理器将文件从HDFS复制到本地磁盘，使任务能够访问文件。任务的工作目录和本地缓存建立起符号连接，任务可以本地化访问文件。

节点管理器为缓存中的每个文件各维护一个计数器统计这些文件的被使用情况。当任务即将运行时，该任务所使用的各种文件的对应计数器值加1；当执行完毕，计数器值减1

#### 计数器的作用

包括了任务计数器、作业计数器和用户自定义计数器

**任务计数器**：任务计数器采集任务的相关信息，并把一个作业中所有任务的结果聚集起来

**作业计数器**：由application master维护，记录作业级别的统计量。比如：作业执行中启动map的任务数，或失败的任务数
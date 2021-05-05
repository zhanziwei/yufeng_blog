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

#### MapReduce的计算过程

![preview](https://pic2.zhimg.com/v2-bd1d4a527fba9a9d265cd05e990ed455_r.jpg)

##### Map端

1. 由程序内的InputFormat（默认实现类为TextInputFormat）读取外部数据，它调用RecordReader的read方法来进行读取，返回k，v键值对
2. 读取的k，v键值对传送给map方法，作为其入参来执行用户定义的map逻辑。
3. context.write方法被调用时，outputCollector组件将map方法的输出结果写入到环形缓冲区中。

##### Reduce端

#### Shuffle和排序

map的输出结果写入到环形缓冲区中，默认大小为100MB，默认阈值为0.8，当超过阈值时，则把内容溢写到磁盘，在溢写过程中，map输出继续写到缓冲区，如果缓冲区被填满，则阻塞map直到磁盘过程完成。

在写磁盘之前，按照partitioner分区，并按照key进行排序，有Combiner则在排序后的输出上运行
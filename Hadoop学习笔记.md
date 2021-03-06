# Hadoop框架课程笔记

## 第一部分：大数据简介
### 大数据的定义
```
   大数据是指无法在一定时间范围内用常规软件进行捕捉、管理和处理的数据集合，是需要新处理模式才有更强的决策力、
洞察发现力和流程优化能力、高增长率和多样化的信息资产。
```

### 大数据的特点
+ 大量（Volume）：采集、存储和计算的数据量都比较大
+ 高速（Velocity）：处理速度快
+ 多样（Variety）：数据形式多样化
+ 真实（Veracity）：确保数据的真实性
+ 低价值（Value）：数据价值密度相对较低

## 第二部分：Hadoop简介
Hadoop是一个适合大数据的分布式存储和计算平台。

### Hadoop的特点
+ 扩容能力（Scalable）：Hadoop是在计算机集群内分配数据并完成计算任务，集群可以方便的扩展到以千计个节点
+ 低成本（Economical）：Hadoop通过普通廉价的机器组成服务器集群来分发以及处理数据，以至于成本很低
+ 高效率（Efficient）：Hadoop可以在节点之间动态并行的移动数据，使得速度非常快
+ 可靠性（Rellable）：能自动维护数据的多份复制，并且在任务失败以后能自动的重新部署计算任务

### Hadoop的优缺点
+ 优点：
   1. Hadoop具有存储和处理数据能力的高可靠性
   2. Hadoop通过可用的计算机集群分配数据，完成存储和计算任务，这些集群可以方便地扩展到数以千计的节点中，具有高扩展性。
   3. Hadoop能够在节点之间进行动态地移动数据，并保证各个节点的动态平衡，处理速度非常快，具有高效性。
   4. Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分配，具有高容错性。
    
+ 缺点：
   1. Hadoop不适用于低延迟数据访问  
   2. Hadoop不能高效存储大量小文件
   3. Hadoop不支持多用户写入并任意修改文件
  
## 第三部分 Apache Hadoop的重要组成
Hadoop = HDFS（分布式文件系统）+ MapReduce（分布式计算框架）+ Yarn（资源调用框架）+ Common模块
+ HDFS（Hadoop Distribute File System）一个高可靠，高吞吐量的分布式文件系统
  + NameNode(NN): 存储文件的元数据信息，比如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等
  + SecondNameNode(2NN): 辅助NameNode更好的工作，用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据快照
  + DataNode(DN): 在本地文件系统存储文件块数据，以及块数据的校验

+ MapReduce：一个分布式的离线并行计算框架
```
MapReduce = Map阶段 + Reduce阶段

Map阶段：Map阶段就是“分”的阶段，并行处理输入数据
Reduce阶段：Reduce阶段就是“合”的阶段，对Map阶段结果进行汇总
```  

+ Yarn：作业调度与集群资源管理的框架
  + ResourceManager(RM): 处理客户端请求、启动/监控ApplicationMaster、监控NodeManager、资源分配与调度
  + NodeManager(NM): 单个节点上的资源管理、处理来自ResourceManager的命令、处理来自ApplicationMaster的命令
  + ApplicationMaster(AM): 数据切分、为应用程序申请资源，并分配给内部任务、任务监控与容错
  
+ Common：支持其他模块的工具模块（Configuration、RPC、序列化机制、日志操作）

## 第五部分 HDFS分布式文件系统
### 第一节 HDFS简介
HDFS（Hadoop Distribute File System，Hadoop分布式文件系统）是Hadoop核心组成，是分布式存储服务

### 第二节 HDFS的重要概念
```
HDFS通过统一的命名空间目录树来定位文件；另外，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色（分布式本质就是拆分，各司其职）
```

+ 典型的Master/Slave架构
  ```
  HDFS的架构是典型的Master/Slave结构
  HDFS集群往往是一个NameNode（HA架构会有两个NameNode，联邦机制）+ 多个DataNode组成
  ```
  
+ 分块存储（Block机制）
  ```
  HDFS中的文件在物理上是分块存储的，块大的可以通过配置参数来规定
  ```
  
+ 命名空间（NameSpace）
  ```
  HDFS支持传统的层次型，文件组织结构。用户或者应用程序可以创建目录，然后将文件保存在这些目录里。
  文件系统名字空间的层次结构和大多数现有的文件系统类似：用户可以创建、删除、移动或重命名文件。
  
  NameNode负责维护文件系统的命名空间，任何对文件系统名字空间或属性的修改都将被Namenode记录下来
  ```  

+ NameNode元数据管理
  ```
  HDFS的目录结构及文件分块位置信息叫做元数据
  
  NameNode的元数据记录每一个文件所对应的block信息（block的id，以及所在的DataNode节点的信息）
  ```  

+ DataNode数据存储
  ```
  文件各个block的存储由具体的DataNode负责。
  
  一个block会有多个DataNode来存储，DataNode会定时向NameNode来汇报自己持有的block信息
  ```  

+ 副本机制
  ```
  为了容错，文件的所有block都会有副本。每个文件的block大小和副本系数都是可配置的。
  应用程序可以指定某个文件的副本数。副本数可以在文件创建的时候指定，也可以在之后改变。
  ```
  
+ 一次写入，多次读出
  ```
  HDFS是设计成适应一次写入，多次读取的场景，且不支持文件的修改。
  ```

### 第五节 HDFS读写解析
#### 5.2 HDFS读数据流程
1. 客户端通过Distribute FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址并返回给客户端。
2. 客户端挑选一台DataNode（就近原则，然后随机）服务器，然后请求读取数据
3. DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）
4. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件

#### 5.3 HDFS写数据流程
1. 客户端通过Distribute FileSystem向NameNode请求上传文件。
2. NameNode检查目标文件是否已存在，父目录是否存在，NameNode返回是否可以上传。
3. 客户端请求第一个Block上传到哪几个DataNode服务器上
4. NameNode返回接收数据的DataNode节点，分别为dn1,dn2,...,dnN
5. 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，直到通信管道建立完毕为止。
6. 通信管道中的dn1,dn2,...,dnN逐级应答客户端，确定通道建立完毕。
7. 客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到Packet就会传给通信管道中的dn2,...dnN；
   dn1没传一个packet会放入一个确认队列等待确认。
8. 当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block服务。（重复3-7步）   

### 第六节 NN与2NN
#### 6.1 HDFS元数据管理机制
+ 第一阶段：
  + 第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存
  + 客户端对元数据进行增删改的请求
  + NameNode记录操作日志，更新滚动日志
  + NameNode在内存中对数据进行增删改
  
+ 第二阶段：
  + Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否执行检查点操作结果
  + Secondary NameNode请求执行CheckPoint
  + NameNode滚动正在写的Edits日志
  + 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode
  + Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
  + 生成新的镜像文件fsimage.checkpoint
  + 拷贝fsimage.checkpoint到NameNode
  + NameNode将fsimage.checkpoint重新命名成fsimage

### 第七节 NN故障处理
1. 2NN元数据拷贝到NN的节点下，此种方式存在丢失元数据的情况
2. 搭建HDFS的HA集群

## 第六部分 MapReduce编程框架
### 第一节 MapReduce思想
MapReduce使用的思想就是“分而治之”的思想。MapReduce将任务分为两个阶段处理

+ Map阶段：
  ```
  Map阶段的主要作用是“分”，即把复杂的任务分解为若干个“简单的任务”来并行处理。
  Map阶段的这些任务可以并行计算，彼此间没有依赖关系。
  ```
  
+ Reduce阶段：
  ```
  Reduce阶段的主要作用是“合”，即对map阶段的结果进行全局汇总
  ```

### 第五节 MapReduce原理分析
#### 5.1 MapTask运行机制详解
1. 首先，读取数据组件InputFormat（默认TextInputFormat）会通过getSplits方法对输入文件进行逻辑切分规划得到splits，
   有多少个split就对应启动多少个MapTask。split与block默认是一对一的关系
2. 将输入文件切分为split之后，由RecordReader对象（默认LineRecordReader）进行读取，以\n作为分隔符，读取一行数据返回<key,value>
   Key表示每行首字符偏移量，Value表示这一行文本内容
3. 读取split返回<key,value>，进入用户自己实现的Mapper类种，执行用户重写的Map函数。RecordReader读取一次这里就调用一次。
4. 执行Map逻辑，将Map的每条结果通过context.write进行collect数据收集。在collect中，会先对其进行分区处理，默认使用HashPartitioner
5. 然后将数据写入内存，内存中这片区域叫做环形缓冲区，缓冲区的作用是批量收集map结果，减少IO的影响。
   缓冲区其实就是一个环形数组。缓冲区大小默认100M，每当达到阈值时（默认80%）就将数据溢写到磁盘上。
6. 当溢写线程启动后，需要对这80M数据按照Key进行排序，使用的是快速排序，排序是MapReduce模型的默认行为
7. 当MapTask都执行完毕之后，会对溢写文件进行merge，merge使用的是归并排序。
   最终形成一个文件，并且提供这个文件的所有文件，记录每个reduce对应数据的偏移量

#### 5.3 ReduceTask运行机制详解
1. Copy阶段：简单地拉取数据。Reduce进程启动数据Copy线程（Fetcher），通过HTTP方式请求MaptTask获取属于自己的文件
2. Merge阶段：将不同Map端的数据进行Merge，Merge时使用归并排序。
   Merge有三种方式：内存到内存、内存到磁盘、磁盘到磁盘
3. Reduce阶段：对排序后的数据使用Reduce逻辑，最后将Reduce的结果写到HDFS中

#### 5.3 MapReduce并行度
+ Map并行度：与split的个数相同
+ Reduce并行度：用户通过job.setNumReduceTasks(num)设置
  1. 当num = 0时，表示没有Reduce阶段，输出文件数与Map阶段产生的文件数保持一致
  2. 当num数不设置时，默认为1，输出的文件就只有一个
  3. 如果数据分布不均匀，可能在Reduce阶段产生数据倾斜

## 第七部分 YARN资源调度
### 第一节 Yarn架构
+ ResourceManager（rm）：处理客户端请求、启动/监控ApplicationMaster、监控NodeManager、资源分配与调度
+ NodeManager（nm）：单个节点上的资源管理，处理来自ResourceManager的命令、处理来自ApplicationMaster的命令
+ ApplicationMaster（am）：数据切分、为应用程序申请资源，并分配给内部惹怒我、任务监控与容错
+ Container：对任务运行环境的抽象，封装了CPU、内存等多维资源以及环境变量、启动命令等任务运行相关的信息

### 第二节 Yarn任务提交（工作机制）
1. 作业提交
   1. Client调用job.waitForCompletion方法，向集群提交MapReduce作业
   2. Client向RM申请一个作业id
   3. RM给Client返回该job资源的提交路径和作业id
   4. Client提交jar包、切片信息和配置文件到指定的资源提交路径
   5. Client提交完资源后，向RM申请运行MR的AppMaster
  
2. 作业初始化
   1. 当RM收到Client的请求后，将该Job添加到容量调度器中
   2. 某一个空闲的NM领取到该Job
   3. 该NM创建Container，并产生MRAppMaster
   4. MRAppMaster下载Client提交的资源到本地

3. 任务分配
   1. MRAppMaster向RM申请多个运行MapTask的资源
   2. 空闲的NodeManager到RM上领取MapTask任务，并创建容器
  
4. 任务运行
   1. MRAppMaster向接受了MapTask任务的NM发送启动脚本
   2. MRAppMaster等待所有的MapTask运行完毕后，向RM申请容器，运行ReduceTask
   3. ReduceTask向MapTask获取相应分区的数据
   4. 程序运行完毕后，MR会向RM申请注销自己
  
5. 进度和状态更新
   ```
   YARN中的任务将其进度和状态返回给应用管理器，客户端每秒向应用管理器请求进度更新展示给用户
   ```

6. 作业完成
   ```
   客户端每5秒都会通过调用waitForCompletion()来检测作业是否完成。
   作业完成之后应用管理器和Container会清理工作状态。
   作业的信息会被作业历史服务器存储以备之后用户检查
   ```

### 第三节 Yarn调度策略
+ FIFO Scheduler：先进先出调度器（淘汰）
+ Capacity Scheduler：容量调度器，可以拆分多个对列（Hadoop默认使用）
+ Fair Scheduler：公平调度器，job平分集群资源（CDH默认使用）










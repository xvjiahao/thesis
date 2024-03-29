# A spark library for representing HDFS blocks as a set of random sample data blocks
## 用于将HDFS块表示为一组随机样本数据块的Spark库
*****
### 摘要翻译：
  在集群计算系统中，分析大数据是一个具有挑战性的问题，特别是当数据量超出可用的计算资源时。采样是解决此类问题的首选方法。考虑到数据分布的统计特征，它汇总或  减少了数据量。但是，由于需要对整个数据进行全面扫描，因此通过逐条记录绘制来采样海量数据的传统方法是计算量巨大的过程。尽管将海量数据划分为一个数据块，但在计算上更便宜。本文所述软件的主要目的是将HDFS块表示为一组随机样本数据块，这些数据块因此存储在HDFS中。我们的经验结果表明，分区操作的性能在现实世界中是可以接受的。
  
-----

- ### 大数据技术出现的原因
  随着云计算和网络技术的不断发展，每天都会产生大量的数据（TB级的数据也很常见），对于这么大量数据的存储、分析和许多的处理任务成为了我们的挑战。Hadoop和Spark是目前最流行的两种基于集群的应用程序，前者是用于分布式的相关工作，后者则是用于并行处理大量的数据。这些系统的基本思想是将大数据文件顺序切成小文件（通常称为块），并将它们分布在集群的各个节点上，以进行并行处理。
  
 - ### Round Random Partitioning algorithm（圆形随机分割算法RRP）
  它的主要工作是去减少RRP的设计并且实现RRP库，RRP主要包含三个组件：数据生成器（data generator），RRP（round-random partitioner）和大型RRP（massive-RRP），数据生成器组件的目的是生成用于分类和回归的综合数据集，而RRP和大型RRP组件的功能相似，都是用于实现圆形随机分割器的实例。
  - **数据生成器（data generator）**
  
    运行在大型数据集上的机器学习算法的执行需要大量的模拟数据来进行估计，基于这一点，就需要数据生成器为大量的算法生成综合数据集，我们目前考虑的算法主要是比较流行的分类算法（classification）和回归算法（regression），但在将来计划将数据生成用于许多不同的分布式任务上。
  - **RRP: round-random partitioner**
    
    RRP主要的作用是将一个大的数据分成许多不重叠的数据块，也就是说每一个数据块都可以视为整个文件的随机样本。下图为使用Apache Spark的RRP的实现步骤：<br>
    ![Fig1.The implementation steps of RRP using Apache Spark](https://ars.els-cdn.com/content/image/1-s2.0-S0167642319300942-gr001.jpg)<br>
    具体步骤：<br>
    * 一个存储在HDFS系统中的大文件D，它有P个块，使用SparkContext.textFile(...)操作，将文件D导入到RDD，并将每个HDFS块映射到RDD分区。
    * 对于每一个RDD分区来说，每一个记录的值都是以<key, value>对的形式保存的，key = 1,2,3...n，其中n代表了当前的分区中有多少条记录。通过转换方法 RDD.mapPartitions(...)实现。<br> 
    * 在将RDD中的值进行map操作之后，可以使用 HashPartitioner(Q)对已完成的RDD进行重新分区。实际上hash函数将 key mod Q的结果作为记录重新分配给新的RDD分区作为索引。<br>
    * 最后从生成的RDD中获取到最终的结果，并且必须注意的是此操作的类型是具有惰性求值的转换操作，这就需要 RDD.saveAsTextFile(...)对数据进行保存<br>
    
  - **Massive-RRP: round-random partitioner for massive data**  <br><br>
    如果数据超出了有限的资源，使用大型的RRP组件解决方案就会比较方便，下图为大型RRP组件的实现步骤：<br>
    ![Fig2.Illustration of massive-RRP: it consists of two stages of randomization to generate the random sample data blocks.](https://ars.els-cdn.com/content/image/1-s2.0-S0167642319300942-gr002.jpg) <br></br>
    ***Stage 1*** <br>
    * 将文件分布存入d个文件夹里，用户可以规定d的值从而确定当前可用的资源数量，每一个文件夹包含 P/d 个块（P为文件数量，d是用户规定的文件夹数量），使用的是均等无替换的随机选择策略。<br>
    * 在每个文件夹下通过应用RRP生成一个新的文件夹（包含P/d个文件），代表了随机样本数据块。<br><br>
    
    ***Stage 2***<br>
    
    * 从上一步中得到的结果中重新进行分布生成以生成其他文件夹，每一个文件夹都有P/d个文件（从上一步结果文件夹中随机选择的，无替换）。<br>+ 在每一个文件夹上应用RRP策略生成新的文件夹（包含 Q/d 个文件，代表随机样本数据块）。<br>
    * 最后，在一个文件夹目录下收集所有文件，形成一个包含Q个文件的数据集（每个文件都是整个数据的随机样本）。<br>
    <br>
    
    ### 总结翻译：
    RRPLib库是一个现成的工具，可以将大数据表示为一组随机样本数据块。实验结果表明，分区操作的执行时间在现实世界中是可以接受的。该系统的显着影响可以通过分析多个数据中心上的分布式数据，以及在基于重采样的统计推断方法中基于块的采样来实现。分析多个数据中心上的分布式数据是一个复杂的过程，尤其是对于海量数据。该库将一个不同数据的块从单个块引入到新的随机块。可以将这个新块视为整个分布式数据的随机样本。此外，在不同的基于重采样的方法中选择块作为子样本可以增强并加快数据处理和分析的性能。


     
    
    


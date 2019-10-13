# Hadoop+: Modeling and Evaluating the Heterogeneity for MapReduce Applications in Heterogeneous Clusters

## 作者：Wenting He *†, Huimin Cui *, Binbin Lu *, Jiacheng Zhao *†, Shengmei Li *, Gong Ruan *†, Jingling Xue ¶, Xiaobing Feng *, Wensen Yang +, Youliang Yan +

<br>

<h2><font face="STCAIYUN" color="#0099ff">摘要翻译：</font></h2>

<font face="微软雅黑" color="STCAIYUN">  尽管异构集群在现代数据中心中得到了广泛采用，但是异构性仍然是一个巨大的挑战，特别是对于大型MapReduce应用程序而言。 在CPU / GPU混合异构集群中，将更多的计算资源分配给MapReduce应用程序并不总是意味着更好的性能，因为同时运行CPU和GPU任务需要争用共享资源。
<br>
  本文提出了一种异构模型，用于在分配异构计算资源（例如CPU和GPU）时预测MapReduce应用程序的共享运行任务之间的共享资源争用。 为了支持该方法，我们提出了一个异构MapReduce框架<font color="black">**Hadoop** +</font>，该框架使CPU和GPU能够协调处理大数据，并利用异构模型来帮助用户实现不同目的的计算资源。<br>
  我们的实验结果显示了三个优点。 首先，Hadoop +利用GPU功能，单独运行时，相对于Hadoop，针对5个实际应用程序的速度提高了1.4倍至16.1倍。 其次，多个同时运行的MapReduce应用程序之间的异构GPU，当多个应用程序同时运行时，速度提高了36.9％（平均17.6％）。 第三，该模型被证明是最佳的或最具成本效益的资源消耗。
</font>

<h2><font face="STCAIYUN" color="#0099ff">一、介绍（INTRODUCTION ）
</font></h2>

<font color="#FF0099" size="4dp">**异构集群的提出：**</font><br>
<font face="微软雅黑">  目前来看MapReduce已经广泛应用于许多领域，包括非计算包括非计算机密集型领域（例如日志分析）和计算机密集型领域（例如深度学习）。为了有效地支持这些多样化的应用，异构集群被广泛应用于数据中心，以在性能和能耗上获得优势。在这样的异构集群中，对于MapReduce应用程序，其性能和成本都会随所消耗的资源而变化。
</font>

<font color="#FF0099" size="4dp">**对于MapReduce异构性面临的挑战：**</font><br>
<font face="微软雅黑">  当CPU和GPU任务同时运行时，资源的抢夺将会带来对性能的影响，以KNN为例来说明这个问题就是当只有一个GPU任务运行时有60MB/s的运行速度，但是当一个CPU和一个GPU同时工作时（CPU+GPU），他的工作效率会下降为53MB/s的速度。这就是因为资源的抢夺会降低GPU的任务的运行速度，并且证明了更多的资源永远不会带来效率的提升，因此，要针对不同的资源对性能进行建模是很有挑战性的。
</font>

<font color="#FF0099" size="4dp">**Hadoop+模型：**</font><br>
<font face="微软雅黑">  为了实现在异构集群中对MapReduce应用程序的异构性进行建模，可以引入异构MapReduce框架Hadoop +，该框架使用户提供的CUDA / OpenCL功能可以插入Hadoop。此外，Hadoop +使用户能够灵活地控制并同时运行GPU和CPU任务，以最大化数据处理速度或选择最有效的配置。
</font>

<br>
<br>
<h2><font face="STCAIYUN" color="#0099ff">二、研究背景和目的（BACKGROUND AND MOTIVATION）
</font></h2>

<font color="#FF0099" size="4dp">**2.1 MapReduce**</font><br>

<font face="微软雅黑">  先了解几个概念
</font><br>

- <font face="微软雅黑">MapReude：MapReduce是Hadoop平台的核心之一，主要进行分布式并行计算，分为Map和Reduce，可以方便地处理海量数据，通过下图示例（求和： 1 + 5 +7 + 3 +4 +9 +3 + 5 +6）可以进行更好地理解。
  </font><br>
  ![](https://img-blog.csdn.net/20181020094407760?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDc4Njg4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)<br><br>
- <font face="微软雅黑">K近邻算法（ K-nearest neighbors）：下面代码块中就是在Hadoop中运行的KNN算法，伪代码中的 ***train*** 代表的是训练样本，***TEST*** 代表测试集，Map将 train 和 TEST集分别作为key和value进行输入，计算每个tarin到TEST集中的每一项的距离，之后Reduce获取到距离的集合选择每一个测试样本最近的topK。
  </font><br>

  ```
    function Map(train,TEST) 
        Compute All Distances(train,TEST) 
    end function 
    function Reduce(test,DistanceOf(test,TRAIN)) 
        Select TopK(Distance(test,TRAIN)) 
    end function
  ```

  <br>

<font color="#FF0099" size="4dp">**2.2 目的实验（Motivation Example）**</font><br>

<font face="微软雅黑" color="pink">***首先重点介绍一下本次实验的一些重点：***</font><br>

- <font>Hadoop +希望在GPU资源可用时在GPU上启动用户提供的地图/减少用CUDA / OpenCL编写的任务。</font>
- <font>Hadoop +希望以与Hadoop相同的方式在一个或多个CPU内核上启动传统的map / reduce任务。 Hadoop映射器或多线程映射器是单线程或多线程映射任务。
  </font><br>
  <br>
  <font face="微软雅黑">使用的平台是Intel的六核Xeon E5-2620芯片，并配置了两个NVIDIA Tesla C2050 GPU。使用数据处理速度的指标来表示应用程序的性能，该公式由以下公式计算：<br>***dps = V/T***<br>
  其中V代表集群中总共需要处理的数据总量，T代表处理数据所需要的时间</font><br>

<font face="微软雅黑" color="pink">***实验测试结果显示：***</font><br>

![](http://m.qpic.cn/psb?/V12dDOqO2iwHZM/FDsvhoqXgYCvcY61bK55frzxxVBXUnmHK95rUW0MuFE!/b/dL4AAAAAAAAA&bo=vAJyAbwCcgEDFzI!&rf=viewer_4)
名词解释：G代表GPU个数， C代表CPU个数， T代表CPU中的线程数量<br>

从图中我们可以得出以下结论：
- 在不适用任何CPU的情况下，使用一个GPU的dps（60MB/s）是低于使用两个GPU的dps（65MB/s）
- 同时使用GPU和CPU会造成资源的抢夺降低dps，如图中的第一条和第三条可以对比出来，仅使用一个GPU（60MB/s）的dps是高于使用一个（GPU+CPU）的dps（55MB/s）
- 当然影响dps的还有CPU的线程数量，CPU线程数量越多，dps越高

</br>

<font face="微软雅黑" color="pink">***实验结果分析：***</font><br>
由下图可以看出CPU和GPU在执行任务时 I/O 流量的区别，红线代表CPU，蓝线代表GPU。CPU任务的I/O流量在执行任务期间几乎保持不变，原因是Hadoop+利用Hadoop中的执行机制执行CPU任务，该任务仅迭代读取一小部分数据并快速对其进行处理，因此I/O流量保持较低水平；相反，对于GPU而言，其I/O流量的变化幅度就比较大，为了获得较高的GPU占用率，GPU任务会读取大量数据，然后将其传输到GPU，然后启动GPU内核进行处理，从而表现出明显的相位行为。但是当GPU任务正在通过其主机线程读取HDFS数据时，I / O流量较高，而当GPU任务正在执行内核时，I / O流量较低。<br>

![](http://a1.qpic.cn/psb?/V12dDOqO2iwHZM/jT3NHbPgjjgDAnP8wtijdSRBBR1xE7BwqxxemVei3jg!/b/dLgAAAAAAAAA&ek=1&kp=1&pt=0&bo=tQLaALUC2gADFzI!&tl=1&vuin=635661878&tm=1570946400&sce=60-3-3&rf=viewer_4)
</br>

同时我们也测试了关于 KNN 的读取数据的速率，通过下图可以看出，当只有一个GPU运行时，数据读取速率达到了72MB/s的高速度，但是当两个GPU同时运行时，数据的读取速率仅有36MB/s，如果再加入CPU任务同时运行的话，速度会更慢，就像图中最后一块，2个GPU和4个单核CPU同时运行时速度达到了最慢。

![](http://m.qpic.cn/psb?/V12dDOqO2iwHZM/Nz.5QlUOswOUUqU.LgydqbEStPXemFAs2oiAuaDQe1Q!/b/dE8BAAAAAAAA&bo=sAIrAbACKwEDFzI!&rf=viewer_4)
</br>

<font face="微软雅黑" color="pink">***实验总结：***</font><br>
&emsp;&emsp;通过此次实验可以看出，对于在异构集群中运行的MapReduce应用程序的异构性进行建模仍然具有一定的挑战性。总结出来有以下几点问题还需要解决：
- <font color="#7B68EE">当我们给应用程序分配计算资源时，什么因素会影响性能的提升？</font>
- <font color="#7B68EE">如何期望一台计算资源如GPU的贡献能够随应用程序而改变</font>
- <font color="#7B68EE">如何为不同的目的（例如，获得最佳性能或最具成本效益）选择应用程序的资源配置？
</font>

<br><br>

<h2><font face="STCAIYUN" color="#0099ff">三、Hadoop+框架</font></h2>



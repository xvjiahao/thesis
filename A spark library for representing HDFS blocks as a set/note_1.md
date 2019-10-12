# Hadoop+: Modeling and Evaluating the Heterogeneity for MapReduce Applications in Heterogeneous Clusters

## 作者：Wenting He *†, Huimin Cui *, Binbin Lu *, Jiacheng Zhao *†, Shengmei Li *, Gong Ruan *†, Jingling Xue ¶, Xiaobing Feng *, Wensen Yang +, Youliang Yan +
{back}
<br>
<h2><font face="STCAIYUN" color="#0099ff">摘要翻译：</font></h2>

<font face="微软雅黑" color="STCAIYUN">&emsp;&emsp;尽管异构集群在现代数据中心中得到了广泛采用，但是异构性仍然是一个巨大的挑战，特别是对于大型MapReduce应用程序而言。 在CPU / GPU混合异构集群中，将更多的计算资源分配给MapReduce应用程序并不总是意味着更好的性能，因为同时运行CPU和GPU任务需要争用共享资源。
<br>
&emsp;&emsp;本文提出了一种异构模型，用于在分配异构计算资源（例如CPU和GPU）时预测MapReduce应用程序的共享运行任务之间的共享资源争用。 为了支持该方法，我们提出了一个异构MapReduce框架<font color="black">**Hadoop** +</font>，该框架使CPU和GPU能够协调处理大数据，并利用异构模型来帮助用户实现不同目的的计算资源。<br>
&emsp;&emsp;我们的实验结果显示了三个优点。 首先，Hadoop +利用GPU功能，单独运行时，相对于Hadoop，针对5个实际应用程序的速度提高了1.4倍至16.1倍。 其次，多个同时运行的MapReduce应用程序之间的异构GPU，当多个应用程序同时运行时，速度提高了36.9％（平均17.6％）。 第三，该模型被证明是最佳的或最具成本效益的资源消耗。
</font>

<h2><font face="STCAIYUN" color="#0099ff">一、介绍（INTRODUCTION ）
</font></h2>

<font color="#FF0099" size="4dp">异构集群的提出：</font><br>
<font face="微软雅黑">&emsp;&emsp;目前来看MapReduce已经广泛应用于许多领域，包括非计算包括非计算机密集型领域（例如日志分析）和计算机密集型领域（例如深度学习）。为了有效地支持这些多样化的应用，异构集群被广泛应用于数据中心，以在性能和能耗上获得优势。在这样的异构集群中，对于MapReduce应用程序，其性能和成本都会随所消耗的资源而变化。
</font>

<font color="#FF0099" size="4dp">对于MapReduce异构性面临的挑战：</font><br>
<font face="微软雅黑">&emsp;&emsp;当CPU和GPU任务同时运行时，资源的抢夺将会带来对性能的影响，以KNN为例来说明这个问题就是当只有一个GPU任务运行时有60MB/s的运行速度，但是当一个CPU和一个GPU同时工作时（CPU+GPU），他的工作效率会下降为53MB/s的速度。这就是因为资源的抢夺会降低GPU的任务的运行速度，并且证明了更多的资源永远不会带来效率的提升，因此，要针对不同的资源对性能进行建模是很有挑战性的。
</font>

<font color="#FF0099" size="4dp">Hadoop+模型：</font><br>
<font face="微软雅黑">&emsp;&emsp;为了实现在异构集群中对MapReduce应用程序的异构性进行建模，可以引入异构MapReduce框架Hadoop +，该框架使用户提供的CUDA / OpenCL功能可以插入Hadoop。此外，Hadoop +使用户能够灵活地控制并同时运行GPU和CPU任务，以最大化数据处理速度或选择最有效的配置。
</font>

<h2><font face="STCAIYUN" color="#0099ff">二、研究背景和目的（BACKGROUND AND MOTIVATION）
</font></h2>

<font color="#FF0099" size="4dp">异构集群的提出：</font><br>
<font face="微软雅黑">&emsp;&emsp;目前来看MapReduce已经广泛应用于许多领域，包括非计算包括非计算机密集型领域（例如日志分析）和计算机密集型领域（例如深度学习）。为了有效地支持这些多样化的应用，异构集群被广泛应用于数据中心，以在性能和能耗上获得优势。在这样的异构集群中，对于MapReduce应用程序，其性能和成本都会随所消耗的资源而变化。
</font>


#HeteroSpark: A Heterogeneous CPU/GPU Spark Platform for Machine Learning Algorithms
<br>
##一、问题的提出
对大数据的各类分析算法都需要大量的计算。Spark是针对这种计算开发出的一个分布式内存计算框架。然而，Spark没有使用GPU来提升计算能力。本文就是开发了这样一个系统，集成在Spark框架中，充分利用GPU的计算能力来提高计算效率。
<br>
##二、主要贡献
作者认为，主要贡献包括以下三个方面：
<br>
1.将GPU计算资源加入到Spark框架，提高了数据并行处理能力和处理速度；
<br>
2.为Spark框架增加了一个即插即用的扩展接口，可以在不修改目前已有程序代码的基础上完成对GPU计算的调用，配置灵活方便；
<br>
3.使用GPU加速在框架内完成，对程序开发人员完全透明。
<br>
文章中作者评估，这种架构比Spark框架在速度上有18倍的提升。
<br>
##三、系统实现
1.系统架构
<br>
作者实现的HeteroSpark框架，在原有的Spark上面增加了一个配置项，配置每个CPU节点与GPU的连接状态，这个状态可以是“local作者事先的HeteroSpark框架，在原有的Spark上面增加了一个配置项，配置每个CPU节点与GPU的连接状态，这个状态可以是“local GPU”、“remote GPU”或“no GPU”。配置项在系统启动的时候读入，运行过程中不可以再更改。
<br>
2.CPU-GPU的通信
<br>
为解决这个问题，作者使用了Java RMI技术，CPU将数据串行化到本地或者远程的GPU JVM端，然后JVM段的RMI服务器反序列化数据，并将数据送入GPU进行计算。虽然使用RMI在序列化和反序列化部分增加了一部分额外的工作，但作者实验发现，这部分工作不会消耗很明显的资源。
<br>
3.两种使用方式
<br>
HeteroSpark框架提供了两种调用模式：
<br>
（1）黑盒模式，用户直接使用HeteroSpark框架就可以了，不需要增加更多的代码。这种模式也可以称作“胶水逻辑”模式。
<br>
（2）开发模式，用户可以在框架中集成自己的加速代码，然后封装调用。
<br>
##四、系统评估
作者对系统通过三个实验样例进行评估：（1）Logistic Regression，使用了Criteo数据集；（2）K-Means，使用MNIST-8M数据集；（3）WordVec，使用Wikipedia页面作为数据集。通过评估作者发现，HeteroSpark系统的“8CPU核 2GPU”相当于128核的Spark效果，“32 CPU核 8GPU”与32核CPU相比，速度增加了18.6倍；与64核CPU相比，速度增加了9倍；与128核CPU相比，速度增加了4倍。

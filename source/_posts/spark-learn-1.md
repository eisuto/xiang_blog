---
title: Spark 学习（1）：久闻大名
date: 2022-01-24 14:44:51
tags: spark 
---
# 简介

Spark 于 2009 年诞生于加州大学伯克利分校 AMPLab，2013 年被捐赠给 Apache 软件基金会，2014 年 2 月成为 Apache 的顶级项目。相对于 MapReduce 的批处理计算，Spark 可以带来上百倍的性能提升，因此它成为继 MapReduce 之后，最为广泛使用的分布式计算框架。
​

# Spark 为什么会出现

1. 为了解决数据量大导致的集群维度上的问题（并行化，容错，资源的动态分配）
1. 为了解决数据流模型在**计算过程中不能高效的共享数据**的问题。MapReduce 计算模型中 Map 的结果必须落盘，然后 Reduce 再读取并加载到内存众的问题，Spark RDD 能够将数据 cache 到内存中，省去了从磁盘加载的过程，同时 Spark shuffle 过程中的数据也是直接放在内存中的，所以 **Spark RDD **在并行计算阶段之间能够高效的共享数据。
1. 为了解决数据容错的问题。一般容错有两种方式: 1.在主机之间复制数据（恢复时间短，但使用内存和磁盘多）2.对各主机的更新情况做日志记录（不需要额外内存，消耗时间长）但这两种方式对于数据密集型的任务来说代价很高。所以** RDD 提供一种基于粗粒度变换的接口**，该接口会将相同的操作应用到多个数据集上。这使得他们可以通过记录用来创建数据集的变换（lineage），而不需存储真正的数据，进而达到高效的容错性。
1. RDD 管理。 Spark提供了三种三种对持久化RDD的存储策略。未序列化Java对象存于内存中、序列化后的数据存于内存、磁盘中。
1. 宽窄依赖

## 与 Hadoop 的比较


| Hadoop       | Spark                                          |
| ------------ | ---------------------------------------------- |
| 类型         | 分布式基础平台, 包含计算, 存储, 调度           |
| 场景         | 大规模数据集上的批处理                         |
| 价格         | 对机器要求低, 便宜                             |
| 编程范式     | Map+Reduce, API 较为底层, 算法适应性差         |
| 数据存储结构 | MapReduce 中间计算结果存在 HDFS 磁盘上, 延迟大 |
| 运行方式     | Task 以进程方式维护, 任务启动慢                |

# 组件

![640.webp](https://tva1.sinaimg.cn/large/005Rbifqly1gyorgrnnfmj30gf06pmyi.jpg) 

**Spark Core**：实现了 Spark 的基本功能，包含 RDD、任务调度、内存管理、错误恢复、与存储系统交互等模块。  
**Spark SQL**：Spark 用来操作结构化数据的程序包。通过 Spark SQL，我们可以使用 SQL 操作数据。  
**Spark Streaming**：Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API。  
**Spark MLlib**：提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。  
**GraphX(图计算)**：Spark 中用于图计算的 API，性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。  
**集群管理器**：Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。  
**Structured Streaming**：处理结构化流,统一了离线和实时的 API。  

# 特点

Apache Spark 具有以下特点：

- 使用先进的 DAG 调度程序，查询优化器和物理执行引擎，以实现性能上的保证；
- 多语言支持，目前支持的有 Java，Scala，Python 和 R；
- 提供了 80 多个高级 API，可以轻松地构建应用程序；
- 支持批处理，流处理和复杂的业务分析；
- 丰富的类库支持：包括 SQL，MLlib，GraphX 和 Spark Streaming 等库，并且可以将它们无缝地进行组合；
- 丰富的部署模式：支持本地模式和自带的集群模式，也支持在 Hadoop，Mesos，Kubernetes 上运行；
- 多数据源支持：支持访问 HDFS，Alluxio，Cassandra，HBase，Hive 以及数百个其他数据源中的数据。

# 集群架构

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyorh6izvdj30gk07yab8.jpg)

| **Term（术语）** | **Meaning（含义）**                                          |
| ---------------- | ------------------------------------------------------------ |
| Application      | Spark 应用程序，由集群上的一个 Driver 节点和多个 Executor 节点组成。 |
| Driver Program   | 主运用程序，该进程运行应用的 main() 方法并且创建 SparkContext |
| Cluster Manager  | 集群资源管理器（例如，Standlone Manager，Mesos，YARN）       |
| Worker Node      | 执行计算任务的工作节点                                       |
| Executor         | 位于工作节点上的应用进程，负责执行计算任务并且将输出数据保存到内存或者磁盘中 |
| Task             | 被发送到 Executor 中的工作单元                               |

## 执行过程

1. 用户程序创建 SparkContext 后，它会连接到集群资源管理器，集群资源管理器会为用户程序分配计算资源，并启动 Executor；
1. Dirver 将计算程序划分为不同的执行阶段和多个 Task，之后将 Task 发送给 Executor；
1. Executor 负责执行 Task，并将执行状态汇报给 Driver，同时也会将当前节点资源的使用情况汇报给集群资源管理器。

相信大家对 Spark 有了一个基本认识，下期我们将分享 Spark Core 中最重要的部分—— **RDD**（弹性分布式数据集）。




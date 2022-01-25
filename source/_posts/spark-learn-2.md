---
title: Spark 学习（2）：RDD
date: 2022-01-25 16:05:56
tags: spark
---
## 简介
RDD 全称为 Resilient Distributed Datasets，是 Spark 最基本的数据抽象，它是只读的、分区记录的集合，支持并行操作，可以由外部数据集或其他 RDD 转换而来，它具有以下特性：

- **分区**（Partitions），一个 RDD 由一个或者多个分区组成。**每个分区会被一个计算任务**所处理，用户可以在创建 RDD 时指定其分区个数，如果没有指定，则默认采用程序所分配到的 CPU 的核心数
- **分区函数**
- **依赖关系**，RDD 会保存彼此间的依赖关系，RDD 的每次转换都会生成一个新的依赖关系，这种 RDD 之间的依赖关系就像流水线一样。在部分分区数据丢失后，可以通过这种依赖关系重新计算丢失的分区数据，而不是对 RDD 的所有分区进行重新计算；
- **分区器**（Partitioner）Key-Value 型的 RDD 还拥有 Partitioner 分区器，用于决定数据被存储在哪个分区中，目前 Spark 中支持HashPartitioner（按照哈希分区)）和 RangeParationer（按照范围进行分区）
- **优先位置列表**，它保存了每个分区的优先位置 (prefered location)。对于一个 HDFS 文件来说，这个列表保存的就是每个分区所在的块的位置，按照“**移动数据不如移动计算**”的理念，Spark 在进行任务调度的时候，会尽可能的将计算任务分配到其所要处理数据块的存储位置。
## 创建 RDD
RDD 有两种创建方式，分别如下：
### 1. 通过现有集合创建
使用 `spark-shell`  以本地模式启动 Spark
```shell
spark-shell --master local[5]
// 相当于
val conf = new SparkConf().setAppName("test").setMaster("local[5]")
val sc = new SparkContext(conf)
```
通过现有集合创建
```scala
val data = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
// 通过集合创建
val dataRDD = sc.parallelize(data) 
// 查看分区数
dataRDD.getNumPartitions
// 指定分区数
val dataRDD = sc.parallelize(data,2)
```
结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/12561498/1643016892244-0ec43a3d-86c8-4851-a0e5-546c024007a6.png#clientId=u24eaea66-5a30-4&from=paste&height=188&id=uec2b3506&margin=%5Bobject%20Object%5D&name=image.png&originHeight=234&originWidth=1002&originalType=binary&ratio=1&size=374069&status=done&style=none&taskId=u8aed304d-3ae0-4131-b9f4-938f2a3e2ca&width=806)
### 2. 通过外部存储创建
可以使用本地文件系统，HBase ，HDFS以及支持 Hadoop InputFormat 的任何数据源。
```scala
val fileRDD = sc.textFile("/usr/test_data/data.txt")
fileRDD.take(1)
```
> 注意：如果在集群环境下从本地文件系统读取数据，则要求该文件必须在集群中所有机器上都存在，且路径相同。

## 操作 RDD
RDD 支持两种操作：

1. _transformations_（转换，从现有数据集创建新数据集）
2. _actions_（在数据集上运行计算后将值返回到驱动程序）

RDD 中的所有转换操作都是惰性的，它们只是记住这些转换操作，但不会立即执行，只有遇到 _action_ 操作后才会真正的进行计算，这类似于函数式编程中的惰性求值。
```shell
scala> val list = List(1,2,3)
list: List[Int] = List(1, 2, 3)

// map 是一个 transformations 操作，foreach 则是一个 actions 操作
scala> sc.parallelize(list).map(_ * 10).foreach(println)
30 tage 0:>                                                          (0 + 6) / 6]
20
10
```
## 缓存 RDD
### 缓存级别
Spark 的速度很快的一个原因是支持缓存，缓存RDD后，如果之后的操作使用了此RDD，则直接从缓存中获取。如果缓存丢失，则通过RDD之间的依赖关系，重新计算丢失分区的数据即可。
Spark 支持多种缓存级别：

| **Storage Level（存储级别）** | **Meaning（含义）** |
| --- | --- |
| MEMORY_ONLY | 默认的缓存级别，将 RDD 以反序列化的 Java 对象的形式存储在 JVM 中。如果内存空间不够，则部分分区数据将不再缓存。 |
| MEMORY_AND_DISK | 将 RDD 以反序列化的 Java 对象的形式存储 JVM 中。如果内存空间不够，将未缓存的分区数据存储到磁盘，在需要使用这些分区时从磁盘读取。 |
| MEMORY_ONLY_SER

 | 将 RDD 以序列化的 Java 对象的形式进行存储（每个分区为一个 byte 数组）。这种方式比反序列化对象节省存储空间，但在读取时会增加 CPU 的计算负担。仅支持 Java 和 Scala 。 |
| MEMORY_AND_DISK_SER

 | 类似于 MEMORY_ONLY_SER，但是溢出的分区数据会存储到磁盘，而不是在用到它们时重新计算。仅支持 Java 和 Scala。 |
| DISK_ONLY | 只在磁盘上缓存 RDD |
| MEMORY_ONLY_2,
MEMORY_AND_DISK_2, etc | 与上面的对应级别功能相同，但是会为每个分区在集群中的两个节点上建立副本。 |
| OFF_HEAP | 与 MEMORY_ONLY_SER 类似，但将数据存储在堆外内存中。这需要启用堆外内存。 |

### 使用缓存
缓存数据有两种方式：

1. persist
1. cache

cache 是 persist 的一种特殊形式，等价于 `persist(StorageLevel.MEMORY_ONLY)`，例如：
```scala
// 存储级别均定义在 StorageLevel 对象中
fileRDD.persist(StorageLevel.MEMORY_AND_DISK)
fileRDD.cache()
```
### 删除缓存
Spark 会自动监视每个节点上的缓存使用情况，并按照最近最少使用（LRU）的规则删除旧数据分区。也可以使用 `RDD.unpersist()` 方法进行手动删除。
## Shuffle 
在 Spark 中，个任务对应一个分区，通常不会跨分区操作数据。但如果遇到 reduceByKey 等操作，Spark 必须从所有分区读取数据，并查找所有键的所有值，然后汇总在一起以计算每个键的最终结果 ，这称为 Shuffle。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/12561498/1643081019869-9dc0d52f-6c80-4adf-8df4-5321128bac80.png#clientId=u471fe126-ffd9-4&from=paste&height=578&id=u8688e911&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1247&originWidth=1733&originalType=url&ratio=1&size=288753&status=done&style=none&taskId=ub34e301a-3cfb-4353-9a49-beb853623a1&width=803)
**Shuffle 是一项昂贵的操作**，因为它通常会跨节点操作数据，这会涉及磁盘 I/O，网络 I/O，和数据序列化。某些 Shuffle 操作还会消耗大量的堆内存，因为它们使用堆内存来临时存储需要网络传输的数据。Shuffle 还会在磁盘上生成大量中间文件，从 Spark 1.3 开始，这些文件将被保留，直到相应的 RDD 不再使用并进行垃圾回收，这样做是为了避免在计算时重复创建 Shuffle 文件。如果应用程序长期保留对这些 RDD 的引用，则垃圾回收可能在很长一段时间后才会发生，这意味着长时间运行的 Spark 作业可能会占用大量磁盘空间，通常可以使用 spark.local.dir 参数来指定这些临时文件的存储目录。
由于 Shuffle 操作对性能的影响比较大，所以需要特别注意使用，以下操作都会导致 Shuffle：

- **涉及到重新分区操作**： 如 repartition 和 coalesce；
- **所有涉及到 ByKey 的操作**：如 groupByKey 和 reduceByKey，但 countByKey 除外；
- **联结操作**：如 cogroup 和 join；
## 宽依赖和窄依赖
RDD 和它的父 RDD(s) 之间的依赖关系分为两种不同的类型：

- **窄依赖 (narrow dependency)**：父 RDDs 的一个分区最多被子 RDDs 一个分区所依赖；
- **宽依赖 (wide dependency)**：父 RDDs 的一个分区可以被子 RDDs 的多个子分区所依赖。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/12561498/1643081313864-3b48a191-ba7b-4b55-b30b-a7dfd7258d0f.png#clientId=u471fe126-ffd9-4&from=paste&height=555&id=uad231af1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=594&originWidth=904&originalType=url&ratio=1&size=134195&status=done&style=none&taskId=u60667b6b-52ea-47b0-a2dc-9a3b4919d89&width=844)
**两种依赖对比：**

| **​**
 | **窄依赖** | **宽依赖** |
| --- | --- | --- |
| 计算方式 | 父分区流水线方式 | 父分区全部计算后 Shuffle |
| 数据恢复 | 重新对丢失分区的父分区进行计算 | 所有父分区数据进行计算并再次 Shuffle |

窄依赖允许在一个集群节点上以流水线的方式（pipeline）对父分区数据进行计算，例如先执行 map 操作，然后执行 filter 操作。
而宽依赖则需要计算好所有父分区的数据，然后再在节点之间进行 Shuffle，这与 MapReduce 类似。
## DAG
RDD(s) 及其之间的依赖关系组成了 DAG(有向无环图)，DAG 定义了这些 RDD(s) 之间的 Lineage(血统) 关系，通过血统关系，如果一个 RDD 的部分或者全部计算结果丢失了，也可以重新进行计算。那么 Spark 是如何根据 DAG 来生成计算任务呢？主要是根据依赖关系的不同将 DAG 划分为不同的计算阶段 (Stage)：

- 对于窄依赖，由于分区的依赖关系是确定的，其转换操作可以在同一个线程执行，所以可以划分到同一个执行阶段；
- 对于宽依赖，由于 Shuffle 的存在，只能在父 RDD(s) 被 Shuffle 处理完成后，才能开始接下来的计算，因此遇到宽依赖就需要重新划分阶段。

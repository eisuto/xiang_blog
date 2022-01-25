---
title: Spark 学习（3）：常用算子
date: 2022-01-25 16:08:14
tags:
---
## Transformation
| **Transformation 算子** | **Meaning（含义）** |
| --- | --- |
| **map**(_func_) | 对原 RDD 中每个元素运用 _func_ 函数，并生成新的 RDD |
| **filter**(_func_) | 对原 RDD 中每个元素使用_func_ 函数进行过滤，并生成新的 RDD |
| **flatMap**(_func_) | 与 map 类似，但是每一个输入的 item 被映射成 0 个或多个输出的 items（ _func_ 返回类型需要为 Seq ）。 |
| **mapPartitions**(_func_) | 与 map 类似，但函数单独在 RDD 的每个分区上运行， _func_函数的类型为 Iterator<T> => Iterator<U> ，其中 T 是 RDD 的类型，即 RDD[T] |
| **mapPartitionsWithIndex**(_func_) | 与 mapPartitions 类似，但 _func_ 类型为 (Int, Iterator<T>) => Iterator<U> ，其中第一个参数为分区索引 |
| **sample**(_withReplacement_, _fraction_, _seed_) | 数据采样，有三个可选参数：设置是否放回（withReplacement）、采样的百分比（_fraction_）、随机数生成器的种子（seed）； |
| **union**(_otherDataset_) | 合并两个 RDD |
| **intersection**(_otherDataset_) | 求两个 RDD 的交集 |
| **distinct**([_numTasks_])) | 去重 |
| **groupByKey**([_numTasks_]) | 按照 key 值进行分区，即在一个 (K, V) 对的 dataset 上调用时，返回一个 (K, Iterable<V>)  
| **reduceByKey**(_func_, [_numTasks_]) | 按照 key 值进行分组，并对分组后的数据执行归约操作。 |
| **aggregateByKey**(_zeroValue_,_numPartitions_)(_seqOp_, _combOp_, [_numTasks_]) | 当调用（K，V）对的数据集时，返回（K，U）对的数据集，其中使用给定的组合函数和 zeroValue 聚合每个键的值。与 groupByKey 类似，reduce 任务的数量可通过第二个参数进行配置。 |
| **sortByKey**([_ascending_], [_numTasks_]) | 按照 key 进行排序，其中的 key 需要实现 Ordered 特质，即可比较 |
| **join**(_otherDataset_, [_numTasks_]) | 在一个 (K, V) 和 (K, W) 类型的 dataset 上调用时，返回一个 (K, (V, W)) pairs 的 dataset，等价于内连接操作。如果想要执行外连接，可以使用 leftOuterJoin, rightOuterJoin 和 fullOuterJoin 等算子。 |
| **cogroup**(_otherDataset_, [_numTasks_]) | 在一个 (K, V) 对的 dataset 上调用时，返回一个 (K, (Iterable<V>, Iterable<W>)) tuples 的 dataset。 |
| **cartesian**(_otherDataset_) | 在一个 T 和 U 类型的 dataset 上调用时，返回一个 (T, U) 类型的 dataset（即笛卡尔积）。 |
| **coalesce**(_numPartitions_) | 将 RDD 中的分区数减少为 numPartitions。 |
| **repartition**(_numPartitions_) | 随机重新调整 RDD 中的数据以创建更多或更少的分区，并在它们之间进行平衡。 |
| **repartitionAndSortWithinPartitions**(_partitioner_) | 根据给定的 partitioner（分区器）对 RDD 进行重新分区，并对分区中的数据按照 key 值进行排序。这比调用 repartition 然后再 sorting（排序）效率更高，因为它可以将排序过程推送到 shuffle 操作所在的机器。 |

## Action
| **Action（动作）** | **Meaning（含义）** |
| --- | --- |
| **reduce**(_func_) | 使用函数_func_执行归约操作 |
| **collect**() | 以一个 array 数组的形式返回 dataset 的所有元素，适用于小结果集。 |
| **count**() | 返回 dataset 中元素的个数。 |
| **first**() | 返回 dataset 中的第一个元素，等价于 take(1)。 |
| **take**(_n_) | 将数据集中的前 _n_ 个元素作为一个 array 数组返回。 |
| **takeSample**(_withReplacement_, _num_, [_seed_]) | 对一个 dataset 进行随机抽样 |
| **takeOrdered**(_n_, _[ordering]_) | 按自然顺序（natural order）或自定义比较器（custom comparator）排序后返回前 _n_ 个元素。只适用于小结果集，因为所有数据都会被加载到驱动程序的内存中进行排序。 |
| **saveAsTextFile**(_path_) | 将 dataset 中的元素以文本文件的形式写入本地文件系统、HDFS 或其它 Hadoop 支持的文件系统中。Spark 将对每个元素调用 toString 方法，将元素转换为文本文件中的一行记录。 |
| **saveAsSequenceFile**(_path_) | 将 dataset 中的元素以 Hadoop SequenceFile 的形式写入到本地文件系统、HDFS 或其它 Hadoop 支持的文件系统中。该操作要求 RDD 中的元素需要实现 Hadoop 的 Writable 接口。对于 Scala 语言而言，它可以将 Spark 中的基本数据类型自动隐式转换为对应 Writable 类型。(目前仅支持 Java and Scala) |
| **saveAsObjectFile**(_path_) | 使用 Java 序列化后存储，可以使用 SparkContext.objectFile() 进行加载。(目前仅支持 Java and Scala) |
| **countByKey**() | 计算每个键出现的次数。 |
| **foreach**(_func_) | 遍历 RDD 中每个元素，并对其执行_fun_函数 |


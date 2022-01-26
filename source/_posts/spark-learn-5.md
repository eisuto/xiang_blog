---
title: Spark 学习（5）：Spark SQL - DataFrame & Dataset
date: 2022-01-26 17:46:38
tags: spark
---
# 一、Spark SQL 简介
Spark SQL 是 Spark 中的一个子模块，主要用于操作结构化数据。它具有以下特点：

- 能够将 SQL 查询与 Spark 程序无缝混合，允许您使用 SQL 或 DataFrame API 对结构化数据进行查询；
- 支持多种开发语言；
- 支持多达上百种的外部数据源，包括 Hive，Avro，Parquet，ORC，JSON 和 JDBC 等；
- 支持 HiveQL 语法以及 Hive SerDes 和 UDF，允许你访问现有的 Hive 仓库；
- 支持标准的 JDBC 和 ODBC 连接；
- 支持优化器，列式存储和代码生成等特性；
- 支持扩展并能保证容错。

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyr7zxqha3j30k906t76i.jpg)
# 二、DataFrame
为了支持结构化数据的处理，Spark SQL 提供了新的数据结构 DataFrame。DataFrame 是一个由具名列组成的数据集。可以理解为数据库中的表。由于 Spark SQL 支持多种语言的开发，所以每种语言都定义了 DataFrame 的抽象，主要如下：
> DataFrame 被标记为 Untyped API，而 DataSet 被标记为 Typed API ，见下文解释

| **语言** | **主要抽象** |
| --- | --- |
| Scala | Dataset[T] & DataFrame (Dataset[Row] 的别名) |
| Java | Dataset[T] |
| Python | DataFrame |
| R | DataFrame |

## 2.1 对比 RDDs
从名字上我们也能知道，DataFrame 面向机构化数据，而RDD面向非结构化数据。  

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyr7zlcstij30go08sq5r.jpg)


# 三、DataSet
Dataset 也是分布式的数据集合，在 Spark 1.6 版本被引入，它集成了 RDD 和 DataFrame 的优点，具备强类型的特点，同时支持 Lambda 函数，但只能在 Scala 和 Java 语言中使用。
在 Spark 2.0 后，为了方便开发者，Spark 将 DataFrame 和 Dataset 的 API 融合到一起，提供了结构化的 API (**Structured API**)，即用户可以通过一套标准的 API 就能完成对两者的操作


## 3.1 静态类型与运行时类型安全
静态类型 (Static-typing) 与运行时类型安全 (runtime type-safety) 主要表现如下:

- 在实际使用中，如果你用的是 Spark SQL 的查询语句，则直到运行时你才会发现有语法错误。
- 而如果你用的是 DataFrame 和 Dataset，则在编译时就可以发现错误 。

DataFrame 和 Dataset 主要区别在于：

- 在 DataFrame 中，当你调用了 API 之外的函数，编译器就会报错，但如果你使用了一个不存在的字段名字，编译器依然无法发现。
- 而 Dataset 的 API 都是用 Lambda 函数和 JVM 类型对象表示的，所有不匹配的类型参数在编译时就会被发现。



我们可以通过以上表现，画出一张**类型安全图谱**：  

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyr801nhsaj31e40l0qaz.jpg)
> Dataset 虽然最严格，但是对开发者来说效率最高

## 3.2 Typed & Untyped
上面提到过， DataFrame API 被标记为 `Untyped API`，而 DataSet API 被标记为 `Typed API`

- 对于语言或 API 而言，虽然 DataFrame 的列类型都是确定的，但是其信息是由 Spark维护的，**只会在运行时检查这些类型与指定类型是否一致**。所以在 Spark 2.0 之后，官方推荐把 DataFrame 看做是 DatSet[Row]，Row 是 Spark 中定义的一个 trait，其子类中封装了列字段的信息。
- DataSet 的类型**由 Case Class(Scala) 或者 Java Bean(Java) 来明确指定的**，在这里即每一行数据代表一个 Person，这些信息由 JVM 来保证正确性，所以字段名错误和类型错误在编译的时候就会被 IDE 所发现。
# 四、总结

- RDDs 适合非结构化数据的处理，而 DataFrame & DataSet 更适合结构化数据和半结构化的处理；
- DataFrame & DataSet 可以通过统一的 Structured API 进行访问，而 RDDs 则更适合函数式编程的场景；
- 相比于 DataFrame 而言，DataSet 是强类型的 (Typed)，有着更为严格的静态类型检查；
- DataSets、DataFrames、SQL 的底层都依赖了 RDDs API，并对外提供结构化的访问接口。
# 五、Spark SQL的运行原理
DataFrame、DataSet 和 Spark SQL 的实际执行流程都是相同的：

1. 进行 DataFrame/Dataset/SQL 编程；
1. 如果是有效的代码，即代码没有编译错误，Spark 会将其转换为一个逻辑计划；
1. Spark 将此逻辑计划转换为物理计划，同时进行代码优化；
1. Spark 然后在集群上执行这个物理计划 (基于 RDD 操作) 。
### 5.1 逻辑计划(Logical Plan)
执行的第一个阶段是将用户代码转换成一个逻辑计划。它首先将用户代码转换成 unresolved logical plan(未解决的逻辑计划)，之所以这个计划是未解决的，是因为尽管您的代码在语法上是正确的，但是它引用的表或列可能不存在。 Spark 使用 analyzer(分析器) 基于 catalog(存储的所有表和 DataFrames 的信息) 进行解析。解析失败则拒绝执行，解析成功则将结果传给 Catalyst 优化器 (Catalyst Optimizer)，优化器是一组规则的集合，用于优化逻辑计划，通过谓词下推等方式进行优化，最终输出优化后的逻辑执行计划。


### 5.2 物理计划(Physical Plan)
得到优化后的逻辑计划后，Spark 就开始了物理计划过程。 它通过生成不同的物理执行策略，并通过成本模型来比较它们，从而选择一个最优的物理计划在集群上面执行的。物理规划的输出结果是一系列的 RDDs 和转换关系 (transformations)。


### 5.3 执行
在选择一个物理计划后，Spark 运行其 RDDs 代码，并在运行时执行进一步的优化，生成本地 Java 字节码，最后将运行结果返回给用户。

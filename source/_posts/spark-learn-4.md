---
title: Spark 学习（4）：累加器与广播变量
date: 2022-01-26 17:46:33
tags: spark
---
# 一、累加器
注意用来对信息进行聚合，主要用于累计计数等场景
## 1.1 为什么要使用累加器？
先看这样一个场景
```scala
var counter = 0
val data = Array(1, 2, 3, 4, 5)
sc.parallelize(data).foreach(x => counter += x)
println(counter)
```
最后counter的结果是0，并不是预期值的15，造成这个问题的原因是闭包
## 1.2 闭包

**Scala 中的闭包：**
```scala
var more = 10
val addMore = (x: Int) => x + more
```
addMore 函数中有两个变量：x 和 more

- **x** 是一个绑定变量，因为是函数的入参，
- **more **是一个自由变量

按照定义：在创建函数时，如果需要捕获自由变量，那么包含指向被捕获变量的引用的函数就被称为闭包函数。
​

**Spark 中的闭包：**
在计算时，，Spark 会将对 RDD 操作分解为 Task，Task 运行在 Worker Node 上。在执行之前，Spark 会对任务进行闭包，如果闭包内涉及到自由变量，则程序会进行拷贝，并将副本变量放在闭包中，之后闭包被序列化并发送给每个执行者。
> 所以，当在 foreach 函数中引用 counter 时，它将不再是 Driver 节点上的 counter，而是闭包中的副本 counter，默认情况下，副本 counter 更新后的值不会回传到 Driver，所以 counter 的最终值仍然为零。

累加器的原理实际上很简单：就是将每个副本变量的最终值传回 Driver，由 Driver 聚合后得到最终值，并更新原始变量。

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyr8084s9lj30gk07yab8.jpg)
## 1.2 使用累加器
SparkContext 中定义了创建累加器的方法
​

执行结果如下：
```powershell
scala> val data = Array(1, 2, 3, 4, 5)
data: Array[Int] = Array(1, 2, 3, 4, 5)

scala> val accum = sc.longAccumulator("My Accumulator")
accum: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 25, name: Some(My Accumulator), value: 0)

scala> sc.parallelize(data).foreach(x => accum.add(x))

scala> accum.value
res3: Long = 15
```
# 二、广播变量
因为闭包的过程中，每个 Task 任务的闭包都会持有自由变量的副本，当 Task 任务很多的情况下，必然会对网络IO造成压力，所以 Spark 为了解决1这个问题，提供了广播变量。
广播变量其实就是：不把自由变量的副本分发到每个任务中，而是分发到每个 Executor，其中的 Task 共享一个副本变量。
```scala
// 定义一个广播变量
val broadcastVar = sc.broadcast(Array(1, 2, 3, 4, 5))
// 使用时，有限使用广播变量，而不是原来的值
sc.parallelize(broadcastVar.value).map(_ * 10).collect()
```

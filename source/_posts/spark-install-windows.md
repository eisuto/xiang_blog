---
title: Spark 在 Windows下的单机环境搭建
date: 2022-01-24 14:05:29
tags: spark 
---
# 安装 Spark
1. 在 [Apache Spark](http://spark.apache.org/downloads.html) 官网下载预编译的文件（ 推荐版本 3.1.2 ）[点此直接下载](https://dlcdn.apache.org/spark/spark-3.1.2/spark-3.1.2-bin-hadoop3.2.tgz) ，解压到某个路径下。如：`G:\spark`
1. 添加环境变量 `SPARK_HOME` => `G:\spark`
1. 添加环境变量 `SPARK_LOCAL_HOSTNAME` => `localhost`
1. 在环境变量 `Path` 中增加值 `%SPARK_HOME%\bin` 和 `%SPARK_HOME%\sbin`
1. 将安装目录下 `/conf` 中的 `log4j.properties.template` 拷贝一份，并进行如下修改，保存后并重命名为 `log4j.properties`
```java
# log4j.rootCategory=INFO, console
log4j.rootCategory=WARN, console
```

6. 将安装目录下 `/conf` 中的 `spark-env.sh.template` 拷贝一份，并添加如下行，保存后重命名为 `spark-env.sh`
```
SPARK_LOCAL_IP = 127.0.0.1
```
# 安装 Hadoop

1. 到 [Apache Hadoop](https://hadoop.apache.org/releases.html) 官网下载预编译文件（为了对应上文Spark，选择3.2.2）[点此直接下载](https://dlcdn.apache.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz) ，并解压到某个路径下。如：`G:\hadoop`
1. 添加环境变量 `HADOOP_HOME`=> `G:\hadoop`
1. 在环境变量 `Path` 中增加值 `%HADOOP_HOME\bin` 和 `%HADOOP_HOME\sbin`
1. 进入安装目录下的 `\etc\hadoop` 文件夹，修改 `hadoop-env.cmd` 文件中的 Java 路径为本机 Java 所在路径。如：
```java
@rem The java implementation to use.  Required.
// 如果是安装在 Program Files 文件夹需使用 PROGRA~1 替换
set JAVA_HOME=C:\PROGRA~1\Java\jdk1.8.0_261
```

5. 下载对应版本的 [winutils](https://github.com/cdarlint/winutils) bin文件，并用下载的文件替换 `\hadoop-3.2.2\bin` 中的对应文件。[点此直接下载](https://downgit.evecalm.com/#/home?url=https://github.com/cdarlint/winutils/tree/master/hadoop-3.2.2/bin)
5. 在C盘中新建文件夹 C: \tmp\hive，并在hadooop目录下，执行以下操作
```java
winutils.exe chmod -R 777 C:\tmp\hive
```

7. 在命令行输入 `hadoop versio` 验证 Hadoop 安装是否成功

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyoqppya2ej30pm04a423.jpg)

8. 在命令行输入 `spark-shell` 验证spark启动是否成功，随后在 `4040` 端口，即可访问Spark UI

![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyoqq9iiuej30u90bgn6s.jpg)
![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1gyoqqdwayyj31h60ecwix.jpg)

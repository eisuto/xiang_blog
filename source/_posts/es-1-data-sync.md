---
title: ES 学习（1）：使用 logstach 将 mysql 数据导入到 elasticSearch 中
date: 2022-07-22 15:23:09
tags: elasticsearch,logstash,mysql
---

# 1. 软件准备及版本
- elasticsearch 6.2.4
  - kibana 6.2.4
  - elasticsearch-head 或 Multi Elasticsearch Head
- logstash 6.2.4
- mysql 8.0 / 5.4

# 2. 配置 logstash
1. 在 logstash 根目录下 建立 `mysqletc` 文件夹
2. 创建 `channel.sql` 文件  
填写你想到入的表即可
    ```sql
    SELECT * FROM shy_article
    ```
3. 放入 mysql 驱动包  
   [点我下载](https://downloads.mysql.com/archives/c-j/)
4. 创建 `mysql.conf` 文件  
    ```javascript
    input {
            stdin {}
            jdbc {
                    # mysql连接驱动包
                    jdbc_driver_library => "G:\usr\logstash-6.2.4\mysqletc\mysql-connector-java-8.0.28.jar"
                    # mysql连接驱动类
                    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
                    # mysql地址
                    jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/shmily_search"
                    # 时区
                    jdbc_default_timezone => "Asia/Shanghai"
                    # 用户名
                    jdbc_user => "root"
                    # 密码
                    jdbc_password => "233333"
                    # 开启分页
                    jdbc_paging_enabled => "true"
                    # 分页大小
                    jdbc_page_size => "1000"
                    # 开启大小写敏感
                    lowercase_column_names => "false"
                    # 同时时间间隔，分 时 月 周 年 时区
                    schedule => "*/5 * * * * Asia/Shanghai"
                    # 读取数据库的 sql 文件
                    statement_filepath => "G:\usr\logstash-6.2.4\mysqletc\channel.sql"
            }
    }
    filter {
            json {
                source => "message"
                remove_field => ["message"]
            }
        }
        output {
            elasticsearch {
                # es 地址
                hosts => ["localhost:9200"]
                # 使用 mysql 表的 id 作为文档id
                document_id => "%{id}"
                # 索引名称
                index => "shmily"
                # type
                document_type => "doc"
            }
        }
    ```
5. 设置自动创建索引  
在 es5 以上版本，需要设置自动创建索引，所以我们在 kibana 中 执行：
    ```json
    PUT /_cluster/settings
    {
        "persistent" : {
            "action": {
            "auto_create_index": "true"
            }
        }
    }
    ```
6. 开始导入  
在 logstash/bin 目录下 执行：`logstash -f ../mysqletc/mysql.conf`   
当出现 `Pipelines running {:count=>1, :pipelines=>["main"]}` 表示管道开启成功  
等待一会，出现如下输出后，表示导入成功
```shell
(0.015714s) SELECT version()
(0.639780s) SELECT count(*) AS `count` FROM (SELECT * FROM shy_article) AS `t1` LIMIT 1
(0.182102s) SELECT * FROM (SELECT * FROM shy_article) AS `t1` LIMIT 1000 OFFSET 0
```
7. 查看结果
打开 elasticsearch-head 选择我们创建的索引，即可看到导入的数据
![image.png](https://tva1.sinaimg.cn/large/005Rbifqly1h4frmbxvpxj31gg0nnnpd.jpg)
### 插入一条语句

```
INSERT INTO rules (classify,product_url,manufacture,rule_name,rule_content,product_country,product_des) VALUES('路由器','https://www.cambiumnetworks.com/products/wifi/cnpilot-r195p-wi-fi-router/','Cambium Networks, Inc.','cnPilot-R195P-4.7-R9','banner="cnPilot R195P 4.7-R9"','国外','r195P为VoIP提供两个ATA端口，为Cambium PMP或ePMP固定无线用户模块提供Cambium 30 V电源。') 
```

#### 插入多条语句可进行value后进行逗号进行追加，或者利用语言进行数据库连接后批量插入（如：利用JDBC）

### 查询语句

> 查询指定分类下规则

```
SELECT classify,product_url,manufacture,rule_name,rule_content,product_des FROM rules WHERE classify='路由器'
```

### 更新语句

> 更新厂商

```
UPDATE rules SET manufacture = '测试的厂商'
```

### 删除语句

> 删除插入规则

```
DELETE FROM rules WHERE id = 1
```

### 如何开启慢查询日志，如何分析

1.查看是否开启慢查询日志

```
show variables like 'slow_query_log'
```

2.查看慢查询日志位置

```
show variables like 'slow_query_log_file'
```

3.开启慢查询日志

```
set global slow_query_log=on; 
```

4.查询执行次数最多的10条慢查询

```
mysqldumpslow -s c -t 10  /var/lib/mysql/iZ8vbis43wevef242spbulZ-slow.log
```

5.设置慢日志查询时间

```
set global long_query_time=4
```

注意：退出此次连接后再次连接可生效

6.查看有几条慢查询日志

```
show global status like '%slow_queries%';
```

### 分析流程

开启慢查询日志，设置超过几秒为慢SQL语句，抓取慢SQL语句；通过explain查看执行计划，对慢SQL语句分析；创建索引并调整语句，再查看执行计划，对比调优结果。

### explain 分析参数

[expain参数]: https://www.cnblogs.com/Zhangjiaqibk/p/15111190.html

> 首先关注查询类型type列，如果出现all关键字，代表全表扫描，没有用到任何index；再看key列，如果key列是NULL，代表没有使用索引；然后看rows列，该列数值越大意味着需要扫描的行数越多，相应耗时越长；最后看Extra列，要避免出现Using filesort或Using temporary这样的字眼，这是很影响性能的。
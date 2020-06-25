# #隶属部门及处理数据库人员规模

这个问题可能是询问你们公司的规模

# #公司的主要业务



# #通过数据库的集群和实例，描述一下公司的业务架构

业务架构即为：应用直接去DB取数据，部分登陆信息会缓存到redis中

# #mysql集群的模式MHA或者其他类型

这个问题需要再看看书，争取写一个innodb cluster 或者MHA

# #mysql版本5.7和5.6和8的区别？

该问题主要考察在某些情况下使用哪个版本的数据库

5.7的新特性：(以下特性主要来源于mysql官网白皮书，还是中文的，建议阅读。下载链接如下https://www.mysql.com/cn/why-mysql/white-papers/whats-new-mysql-5-7-zh/)

- 性能提升，mysql5.7的基准测试可以支持160w的qps，这个是mysql5.6的3倍

- 原生json的支持

- Performance Schema ，提供了更多的监控项

- SYS Schema 新增了一个schema，主要是information.shcema和Performance.shecma的集合。

- innodb在线修改，在线修改innodb buffer pool的大小，在线修改索引名，在线修改vachar的列长

- innodb缓存保留，在你重启mysql后，innodb缓冲池会保存你25%最热的数据，这样就不需要预热

- 优化器相关，反正就是很厉害，因为我也看不懂

- json explain 改进了这个命令的输出值，增加了资源消耗预估值等信息

- 新的优化器Hints和hint语法

- Generated Columns 虚拟列

- 服务端语句超时，甚至可以在某个sql里面执行，示例如下

  ```sql
  SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM my_table;
  ```

- 多源复制，即一个slave可以从多个master复制binlog的信息

- **基于事务的并行复制**这个特性很重要，命令为

  ```shell
  slaveparallel-type=LOGICAL-CLOCK
  ```

- 在线复制变更，即你可以在线把基于binlog的postion的复制换成GTID方式，而不需要重启mysql
- 半同步复制性能加强
- Group Replication 组复制，类似于mongodb的复制集，具备failover能力。

mysql8.0新特性





#MHA的原理

#高可用行的具体实现，主库挂掉的处理方式

#普罗米修斯是否使用过？

#监控的报警方式？是否有7*24值班

#备份方案，为啥不用xtrabackup 去备份

#redis集群用的是那个版本，有没有用过哨兵

#redis cluster 模式的原理，为什么要搭建在2台机器上？2个节点不能实现高可用，起码需要四个节点

该问题我理解是需要两个节点做haproxy，做一个轮询，failover之类的。

#使用了解oracle，有木有做过linux下的oracle的安装？

#是否熟悉hive
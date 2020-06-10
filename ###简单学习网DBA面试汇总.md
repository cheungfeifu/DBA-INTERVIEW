###简单学习网DBA面试汇总

## 1.单实例MySQL数据量？

这个最好在MySQL目录下du -h一下看看。

## 2.你们的MySQL集群的模式，一主多从/多主多从 ，假如是双主双从，你们是单写还是双写?

MySQL双主双从方案中，我们是只写某一个主库，然后同步到另一个主库，然后再同步到主库对应的从库

## 3.高可用方案，代理节点挂掉后的处理方式？haproxy单点故障后如何处理？了解haproxy的原理吗？

这个问题是我描述双主双从的架构时，在读取从库的时候，使用haproxy进行从库的代理，进行负载均衡，但是假如代理节点挂掉后，整个读取从库也会挂掉，这个不符合高可用。对应方案可能需要学习一下Atlas 或者其他MySQL的中间件

haproxy的原理

这个看不懂

reference

https://yq.aliyun.com/articles/462779

https://cloud.tencent.com/developer/article/1349133

## 4.proxysql是否用过？假如不熟练 建议删除

这个技能点删除，不会的不要写，写了回答不上，你就扯淡了。

## 5.zabbix的监控脚本，这里面试官真是傻逼 我没有用脚本去监控MySQL

这个问题面试官是真他妈的狗，我简历没有写使用python写脚本去监控MySQL，我逼逼了五分钟的zabbix，他突然打断我说 不想听zabbix，想知道你是如何使用python写脚本去监控的。

## 6.如何判断去查看数据库是否正常，就是数据库的那些指标？

- 数据库是否挂掉，直接在服务器尝试登录一下

- 查看mysql.user 那些host是%，因为%是任何IP都可以登录的

- grant  all  on 是否这些用户需要全部权限？

- 连接数：SHOW  full   processlist 查看连接数是否过高

- 查看当前失败连接数 SHOW GLOBAL  STATUS  LIKE  'aborted_connects'

- 查看有多少由于客户没有正确关闭连接而死掉的连接数 SHOW GLOBAL  STATUS  LIKE  'aborted_clients'

- MySQLd.log 直接去里面grep 一下error关键字，看看有那些错误信息需要你处理

- innodb死锁：SHOW  ENGINE  INNODB   STATUS

- 锁等待，元数据锁：元数据锁 在show processlist 会出现 waiting for table metadata lock 锁等待，在information_schema.innodb_trx/locks/lock_waits 会有相关信息

- 配置文件是否有人修改过，/etc/my.cnf 最好写个定时任务，定时备份这个文件

- 慢查询日志 

- 主从异常：在从库SHOW  SLAVE   STATUS 查看IO线程和SQL线程是否都是yes 看seconds_behind_master是否大于0

- 全表扫描 SHOW GLOBAL STATUS LIKE'Handler_read%';

-  SHOW GLOBAL STATUS 中的各类参数需要了解

  reference

  https://blog.csdn.net/devin223/article/details/46455243

## 7.MySQL的版本，有没有做过MySQL的升级？

MySQL5.7，我没有做过升级，我们一直使用MySQL5.7

升级的方法一般有两类：

1.利用mysqldump来直接导出sql文件，导入到新库中，这种方法是最省事儿的，也是最保险的，缺点的话，也显而易见，大库的mysqldump费时费力。

2.直接替换掉mysql的安装目录和my.cnf，利用mysql_upgrade 来完成系统表的升级，这种方法需要备份原有的文件，但属于物理拷贝，速度较快。缺点的话，跨版本升级不推荐这么做，比如mysql5.1升级到mysql5.6,mysql5.5升级到mysql5.7等。

reference

https://segmentfault.com/a/1190000010716561

## 8.Linux搭建MySQL后，会不会对MySQL所在的Linux机器做一下优化？

这个问题很懵逼，因为拿到机器后就是直接yum安装MySQL，具体到MySQL所在的Linux机器的性能优化，我理解不会是设置配置文件的innodb_buffer_size之类的，

因为在看redis cookbook时看到在生产环境使用redis，需要修改Linux的一些参数，例如Linux的内核和操作系统级别的参数，

例如设置 sudo sysctl -w net.core.somaxconn=65535

该参数默认为128

具体优化待定

## 9.对于Linux的熟练程度？

这个问题看自己，例如防火墙的设置，文件系统，yum等等操作

## 10.在使用sysbench对数据库进行压力测试后，你会看那些指标，如何使用这些指标？

- TPS/QPS：衡量吞吐量。

- 响应时间：包括平均响应时间、最小响应时间、最大响应时间、时间百分比等，其中时间百分比参考意义较大，如前95%的请求的最大响应时间。。

- 并发量：同时处理的查询请求的数量。

  ![计算机生成了可选文字: SQLstatlstlcs： queriesperfomed other： total： tranSa10nS： reder： 0nnS： Generalstatistics： totaltime： totalnumberOfevents： Latency{m蚂： avg： 95thpercentile. Threadsfairness： events(avg/stddev)： executiontime(avg/stddev)： [root@localhost一]#0 2b92359 1949g32 52e457 415375g 2b9231{857·35persec.) 41b375g{13878.€2persec.) {€2persec.) (€.@€persec.) 399。e242s 2b9231 1122·74 25·74 47gg724.68 1b2b4·4375／69·94 299.9828/€.@€](file:///C:/Users/YSRD/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

  如上是我在虚拟机做的sysbench测试报告，从上往下一次解释：

  read代表总读取数，write 代表总写入数，other 代表增删改查外的操作，例如commit，total代表总数量

  transactions ：总事务数，后面的867.36代表TPS, queries 代表总查询数量，后面的13878.02代表QPS

  ignored errors:总忽略错误数，reconnects :总重连数量，total time 测试花费时间 total number events 总事务数量，latency 相应时间，注意侧重于95th percentitl 超过95%平均耗时 是25.74毫秒（ms）

  

## 11.MySQL的tps和qps的计算方式？

这个问题我当时知道这两个指标，但是计算方式我讲错了，tps我当时讲的计算方式是查看information_schema中的innodb_trx的数量就是每秒事务数

qps是show processlist中select的数量，这个答案肯定是不对的，正确如下：

QPS：Queries Per Second     查询量/秒，是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理查询量多少的衡量标准

TPS :  Transactions Per Second  是事务数/秒，是一台数据库服务器在单位时间内处理的事务的个数。

计算TPS方式如下：

```sql
-- MySQL启动后到现在的提交事务数
 show global status like 'com_commit';

-- 62500563

-- MySQL启动后到现在的事务回滚数
show global status like 'com_rollback';

-- 45182661

-- MySQL启动后到现在一共过了多少秒
show global status like 'uptime';

-- 1909846
-- 计算TPS 

select (62500563+45182661)/1909846;
```

计算QPS方式如下：

```sql
-- 已经发送给服务器的查询的个数
show global status like 'questions';

-- 182411980

-- MySQL启动后到现在一共过了多少秒
show global status like 'uptime';
-- 1910518

-- 计算QPS

select 182411980/1910518;
```

从这里我们学习到查看一个数据库的状态要仔细看：

Show global status 该命令下的各类参数

reference：

https://www.cnblogs.com/asea123/p/10572766.html

https://www.cnblogs.com/fjping0606/p/5821289.html

## 12.redis集群 有多少个槽，槽的分配的方式？

这个问题是简历有写搭建redis 集群，所以面试官着重问了一下槽

共计16384个槽，槽的分配方式如下：

16384/节点数 = 每个节点对应的槽

具体每个key插入到那个节点则需要如下计算：

HASH_SLOT=SRC16(key) mod 16384



## 13.主键索引和唯一索引的区别？

这个问题当时觉得非常简单，但是回答的很菜

我们直接搜到洋鬼子的答案：

**Primary Key:**

- There can only be one primary key in a table(同一个表只可以有一个主键索引)
- In some DBMS it cannot be `NULL` - e.g. MySQL adds `NOT NULL`（在某些关系行数据库中不能为NULL）
- Primary Key is a unique key identifier of the record（主键是每一行的唯一标记符）

**Unique Key:**

- Can be more than one unique key in one table（一个表可以有多个unique key）
- Unique key can have `NULL` values(unique key 可以为null)
- It can be a candidate key(没看懂)
- Unique key can be `NULL` ; multiple rows can have `NULL` values and therefore may not be considered "unique"（unique key 可以为null，多行包含null，所以不能认为是唯一的）

衍生到另一个问题：MySQL有那些索引，他们的区别？

Mysql常见索引有：主键索引、唯一索引、普通索引、全文索引、组合索引

Mysql各种索引区别：
普通索引：最基本的索引，没有任何限制 普通的 index index_name ( `column` )
唯一索引：与"普通索引"类似，不同的就是：索引列的值必须唯一，但允许有空值。 UNIQUE(唯一索引) 
主键索引：它 是一种特殊的唯一索引，不允许有空值。  PRIMARY KEY ( `column` ) 
全文索引：仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时好空间。没用过
组合索引：为了更多的提高mysql效率可建立组合索引，遵循”最左前缀“原则。ADD INDEX index_name ( `column1`, `column2`, `column3` )  注意最左匹配原则

## 14.percona-toolkit 工具是否用过？ 有没有用过pt-online-schema-change 在线修改表结构？

pt-toolkit是数据库运维经常用到的工具，一定要找出几个你平时用到的给面试官讲

pt-OSC原理：

![img](https://blog-10039692.file.myqcloud.com/1496285991003_5032_1496285991391.png)

pt-online-schema-change使用条件：

1.原表不能存在触发器。

2.原表必须存在主键 PRIMARY KEY 或者 UNIQUE KEY

3.外键的处理需要指定 alter-foreign-keys-method 参数，存在风险

4.在 pt-osc 的执行过程中，如果有对主键的更新操作则会出现重复的数据,见下面链接的 bug 说明:

https://bugs.launchpad.net/percona-toolkit/+bug/1646713

## 15.假如没有用过pt-online-schema-change，那么你们如何在生产环境DDL，以及online ddl的原理？

online ddl 就是在线修改表结构或者增删改查索引，假如没有online ddl的话，则会出现元数据锁的问题，这样对于生成环境是不允许的。

MySQL5.5前DDL的原理示意图如下：

![img](https://blog-10039692.file.myqcloud.com/1496285792905_8514_1496285793592.png)



如上方式，第一需要花费很长的时间，因为copy原数据到新表需要花费很多时间，第二 由于需要创建一个新表，所以对于空间消耗比较大。

MySQL5.7online ddl的操作规则如下：

![img](https://blog-10039692.file.myqcloud.com/1496285940906_3974_1496285941485.png)

5.7online ddl原理如下

![5.7online ddl原理](https://blog-10039692.file.myqcloud.com/1496285951468_1741_1496285951992.png)



5.7 的 Online DDL 使用限制与问题

1.仍然存在排他锁，有锁等待的风险。

2.跟 5.6 一样，增量日志大小是有限制的(由 innodb_online_alter_log_max_size 参数决定大小)

3.有可能造成主从延迟

4.无法暂停，只能中断

reference:

https://cloud.tencent.com/developer/article/1005177

## 16.mongodb是否了解，复制集，复制集的原理？

这个问题真的不会，瞎扯了一顿，直接杀死比赛。


复制集（replica set）是MongoDB用来保持相同的数据集合的一个MongoD进程组，复制集提供了所有生产部署的基础：数据冗余以及高可用。MongoDB的高可用靠的是自动故障转移来实现的，本节就是介绍MongoDB的该部分实现的

个人理解这个复制集类似于MySQL的组复制

复制集最少需要三个节点：

数据主节点：唯一可以写的节点

仲裁节点：

从节点

三个节点的示意图：

![img](https://docs.mongodb.com/manual/_images/replica-set-primary-with-secondary-and-arbiter.bakedsvg.svg)

arbiter不会储存数据，arbiter的主要作用是维持与复制集中所有的其他节点的心跳以保证选举需要的节点数，因为arbiter不是一个数据存储集，arbiter可以提供一个比全功能副本集更廉价的方法来获取法定人数。如果复制集中是偶数个节点，可以通过添加arbiter节点使得primary可以获取到大多数的投票。arbiter不需要专门的硬件支持

假如primary出现故障，转移示意图如下：

![https://docs.mongodb.com/manual/replication/#edge-cases-2-primaries](https://docs.mongodb.com/manual/_images/replica-set-trigger-election.bakedsvg.svg)












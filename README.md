# mysqlsplit
## mysql 分库分表

![](https://i.imgur.com/amWjT1O.jpg)

<pre>
   基本分库分表:
       1:分库分表
       2:分库表冗余
       3:分区表
   分布式事务
       1:XA分布式事务
       2:TCC分布式事务
       3:消息分布式事务
</pre>

<pre>
   Mycat分片规则
   Mycat读写分离
   Mycat故障切换
   Mycat+Percona+Haproxy+keepalived
   Zookeeper搭建Mycat高可用集群
   Mycat注解技术
   Mycat性能监控
   Mycat架构剖析
     1) Mycat线程架构
     2) Mycat网络IO架构
     3) Mycat内存还礼与缓存架构
     5) Mycat连接池 主从切换
   Mycat核心技术
     1）Mycat路由的实现
     2）Mycat跨库join的实现
     3）Mycat数据汇聚和排序
</pre>

![](https://i.imgur.com/LHh2xss.jpg)

Mysql复制的工作方式

![](https://i.imgur.com/gByifSu.jpg)

<pre>
   什么限制了Mysql的性能
      最常见的两个瓶颈：
      1）CPU
	     当数据存储全部存储在内存中，CPU会成为瓶颈
	  2）IO资源
	     一般发生在工作所需的数据远远超过有效内存容量的时候
	   
      硬件性能的优化
         1）低延时
           更快的CPU
         2）高吞吐
           更多的CPU个数,要取决于Mysql版本，不同版本使用CPU的个数是有极限的
      Mysql复制也能在高速CPU下工作非常快，而多CPU对复制帮助却不大，主库上的并发任务传递到备库以后会被简化为串行任务，也就是是备库的瓶颈通常是IO子系统，而不是CPU

      在硬件层面，一个查询可以再执行或者等待，处于等待状态的原因是在运行队列中等待，等待磁盘，等待网络，等待闩(Latch)或者锁(Lock),
         1）如果等待闩或者锁，通常需要更快的CPU
         2）如果在运行队列中等待，那么更多或者更快的CPU都可能有帮助

   1）Cpu架构 CPU核心
      Mysql的扩展模式是指它可以有效利用CPU的数量，以及在压力不断增长的情况下如何扩展，这同时取决于工作负载和系统架构，CPU架构（RISC,CISC,流水线深度），CPU型号，操作系统都影响Mysql的扩展模式

      现代CPU的复杂之处：
         1）频率调整
	     2）boost技术

   2）平衡内存和硬盘资源
      配置大量内存的最大原因其实不是因为可以在内存中保存大量的数据：
         1）最终目的是避免磁盘I/O，因为磁盘I/O比在内存中访问数据要慢的多
	     2）关键是要平衡内存和磁盘的大小。速度，成本和其他因素。
      
      随机I/O，顺序I/O
	     顺序操作的执行速度比随机I/O的操作快，无论是在内存还是在硬盘
         存储引擎执行顺序读比随机读快

      缓存的写入延迟特性：
         1）多次写入，一次刷新
	     2）I/O合并
      预写日志（WAL）策略：采用在内存中变更页面，而不马上刷新到磁盘的策略，后续时机再刷新到磁盘
         缓存命中率实际上也会决定使用多少CPU，所以评估缓存命中率的最好方法是查看CPU的利用率，若CPU使用了99%的时间工作，1%的时间等待，那么缓存的命中率还是不错的。
      
      1）闪存，
      2）固态硬盘，
      3）PCIE存储设备

   3）RAID性能优化,SAN/NAS
      SAN和NAS是两个外部文件存储设备加载到服务器的方法
      SAN基于块访问，使得所有服务器的硬盘像访问一块硬盘
      NAS基于文件的协议来访问

   6）操作系统 文件系统 磁盘调度策略  
      线程  内存交换区
      操作系统选择：
          Linux发行版，最好的策略是使用专为服务器应用程序设计的发行版，而不是桌面版本。
      文件系统：
      通常建议使用XFS文件系统
      选择磁盘调度策略
      内存交换区
      监控操作系统的工具：
          vmstat iostat	

   7）复制
       1）基于语句的复制
       2）基于行的复制
       基于行的复制和基于语句的复制都是通过在主库上记录二进制日志，在备库中重放日志的方式实现异步的数据复制
       低版本的Mysql不能作为高版本Mysql的备库，而高版本的Mysql可以作为低版本Mysql的备库，Mysql具有向后兼容性，高版本的Mysql可以兼容低版本的Mysql

       复制的三步骤
          1）主库把数据更改记录到二进制文件(Binary log)中
             在每次准备提交事务完成数据更新前，主库将数据更新的事件记录到二进制文件中，Mysql会按照事务提交的顺序而非每条语句执行的顺序记录二进制日志记录完二进
             制日志后，主库会告诉粗怒触引擎可以提交事务了。
          2）备库将主库上的日志复制到自己的中继日志(relay Log)中
             备库会启动一个工作线程，成为I/O线程，I/O线程跟主库建立一个普通的客户端连接，然后在主库上启动一个特殊的二进制转储线程，这个二进制转储线程会
	         读取主库上二进制日志中的时间，它不会对事件进行轮训。如果该线程追上了主库，它将进入休眠状态，备库I/O线程会将接收到的事件记录到中继日志中。
          3）备库读取中继日志中的时间，将其重放到备库数据之上

       5）复制事件
       6) 复制问题
          1）数据损坏与丢失
             1）主库意外关闭
             2）备库意外关闭
             3）主库上的二进制文件意外损坏
             5）备库上的中继日志损坏
             6）二进制日志与InnoDB事务日志不同步

          2）使用非事务性表
          3）不确定语句
          5）主库备库使用不同的存储引擎
          6）不唯一的服务器ID
          7）丢失的零时表
          8）Innodb加锁读引起的锁争议
          9）过大的复制延迟
          10）来自主库过大的包
          11）受限的复制带宽
          12）备库发生数据改变
          13) Mysql复制的高级特性

   8）复制拓补结构
       1）一主多备
       2）主动模式下的主主复制，被动模式下的主主复制
       3）环形复制
       5）其他复制方案

   9）分区表、视图 外键约束 游标 全文索引 查询缓存
       分区表是一个独立的逻辑表，底层由多个物理子表组成，创建表时通过partion by子句定义

   10）高性能索引的策略
       在Mysql中，索引是在存储引擎层而不是在服务层实现的，不同存储引擎的索引的工作方式并不一样，也不是所有的存储引擎都支持所有类型的索引。及时多个存储引擎支持同一种类型的索引，其底层的实现也可能不同。
       1) B-Tree
          大多数的MYSQL引擎都支持B-Tree索引，Archive引擎是一个例外，Archive不支持任何索引。
          
          MyIsam使用前缀压缩技术使得索引更小
          InnoDB则按照原数据格式进行存储
          MyIsam索引通过数据的物理位置引用被索引的行
          Innodb则根据主键引用被索引的行

       2) Hash索引
          只有memory引擎支持hash索引,memory引擎是支持非唯一hash索引的，这在数据库世界里面是比较与众不同的，如果多个列的hash值相同，索引会以连表的方式存放多个记录
          指针到同一个hash条目中。

          因为索引自身只需要存储对应的hash值，所以索引的结构非常紧凑，这也让hash索引查找的速度非常快，然而hash索引也有限制
          1）hash索引数据并不是按照索引值顺序存储的，所以也就无法用于排序
          2）hash索引页不支持部分索引列匹配查找。
          3）hash索引只支持比较查询，不支持范围查询
          5）hash索引存在hash冲突
          6）hash冲突如果很多的话，维护索引的代价也很高

          InnoDB引擎有一个特殊的功能叫做"自适应hash索引"，当InnoDB注意到某些索引值被使用得非常频繁时，它会在内存中基于B-Tree索引之上再创建一个hash索引，这样就让B-Tree索引也具有hash索引的一些优点，比如快速的hash查找 ，这是完全自动的内部的行为，用户无法感知或者配置，用户也可以关闭该功能。

       3) R-Tree空间数据索引
          MyIsam表支持空间索引，可以用作地理数据存储，和B-Tree索引不同，这类索引无需前缀索引。空间索引会从所有维度来索引数据。查询时，可以有效使用任意维度来组合查询。

       1）独立的列
       2）前缀索引，索引选择性
       3）多列索引
       5）选择合适的索引列顺序
       6）聚簇索引
       7）覆盖索引
       8）使用索引扫描来排序
       9）压缩索引
       10）冗余和重复索引
       11）索引和锁
   11）Schema与数据类型优化
   12）服务器性能剖析
   13）Mysql查询性能优化
       1）慢查询优化
       2）重构查询
          1）切分查询
          2）分解关联查询
   15) Mysql的存储引擎
       1) Innodb
       2) MyIsam
       3) Archive
       5) memory
       6) NDB集群引擎 
   16) Mysql的锁粒度
   17) Mysql的事务隔离级别
   18) 多版本并发控制MVCC
</pre>

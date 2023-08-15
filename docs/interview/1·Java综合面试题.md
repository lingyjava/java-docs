# 1·Java综合面试题

- [1·Java综合面试题](#1java综合面试题)
- [性能](#性能)
  - [性能调优](#性能调优)
- [数据库优化](#数据库优化)
  - [SQL 语句优化](#sql-语句优化)
  - [数据库索引](#数据库索引)
- [MySQL数据库](#mysql数据库)
- [悲观锁、乐观锁](#悲观锁乐观锁)
- [分布式锁](#分布式锁)
- [JVM内存原理](#jvm内存原理)
  - [GC (垃圾回收)机制](#gc-垃圾回收机制)
- [框架原理](#框架原理)
  - [Ioc原理](#ioc原理)
  - [Aop原理](#aop原理)
- [SpringMVC执行流程](#springmvc执行流程)
- [工厂模式、三种单例模式(手写)](#工厂模式三种单例模式手写)
  - [工厂模式实现](#工厂模式实现)
  - [单例模式实现](#单例模式实现)
- [数据类型题](#数据类型题)
- [JSP题](#jsp题)
- [线程题](#线程题)
- [Redis相关的面试题](#redis相关的面试题)
- [消息队列常见面试题](#消息队列常见面试题)

# 性能

指标：并发用户、响应时间、TPS（每秒事务处理个数）
## 性能调优

1. 通讯层面
   1. 采取异步线程通讯模式，比如socket nio异步线程通讯模式，此模式有负责接收消息的线程和负责处理消息的线程，分工协作。Tomcat6.0版本以后对8080端口的通讯监听就是采用的socketnio异步模式。
   2. 采用队列（比如activemq）或者缓存（redis）。处理并发量大的秒杀场景，将所有请求接收放入队列或者缓存，然后从队列或者缓存中获取部分请求处理，处理完一部分再获取一 部分处理，直到秒杀结束（库存不足）。如果秒杀结束，队列中还有消息未处理，那么将这些消息全部返回客户端，返回商品已售完。

2. 项目集群部署
   1. 采用nginx反向代理，将用户请求分发到集群环境中的不同服务器，降低单个服务器的压力，起到负载均衡的作用。
   2. 增加系统的并发处理能力，可以处理更多的用户请求，防止出现因某台服务器宕机而导致业务中断的单点故障。当某台服务器出现故障时，nginx反向代理会做到自动隔离该服务器，不再将请求转发到宕机的服务器。
   3. 因此，还可做到热部署，在上线过程中选择部分服务器重启，保证业务不间断运行。

3. 缓存

项目采用redis做缓存服务器，将查询多修改少的数据放入缓存，查询时直接从缓存中获取，无需通过数据查询，减少IO操作，大大提高的查询的速度，同时也减轻了对数据库的压力。

4. 资源动静分离

将项目中大量的js、Css、html、图片等静态资源单独部署到服务器，然后通过远程连接地址（比如:http://192.168.2.1:8080/picture/dog. jpg）访问，提高页面加载速度，同时减轻应用服务器的压力。

5. 数据库集群部署

防止数据库访问应用的单点故障，减轻数据库访问压力，公司dba工程师会将oracle部署为集群的rac模式，用到两台服务器安装数据库，两台服务器的数据库数据存储在同一块共享磁盘。

6. SOA服务优化

采用dubbo分布式框架，系统直接的接口调用通过dubbo，比如我们项目调用财务系统的支付接口，该接口就是由财务系统将地址发布到dubbo，然后我们写http客户端去调用dubbo，由dubbo将请求转发财务系统。
# 数据库优化
关于数据库优化需要从三个方面考虑：

1. 数据存储分区
   1. 我们的系统用户数据非常多，单纯的为表建立索引不能满足性能的需要，考虑到用户来自不同的地区，因此按省份做了列表分区，不同省的数据存储到不同的数据分区，当查询数据加上省份条件时，只会检索对应分区的数据，大大缩小数据检索的范围，从而提高查询性能。
   2. 范围分区（比如按照日期字段来建立分区，一个季度或者一个月的数据划分为一个区）；
   3. 散列分区（不指定分区条件，oracle 自动将数据平均分配）；
   4. 复合分区（比如:做了分区后，每个分区里面再次做分区，范围-列表，范围-散列）
2. 表索引
   1. 表建立好分区以后，在分区里面添加索引，进一步提高数据的查询速度，项目里用到索引的比如（客户信息表姓名建立btree索引、身份证号建立唯一索引、客户编号建立主键，主键自带唯一索引）（消费表的消费类别建立位图索引） （经常用到的表与表关联条件建立索引） 
   2. 索引可以提高查询速度，索引有一套独立的系统表存储数据与rowid的对应关系（如书本的目录），比如btree索引，它是个树形结构图，有根节点、分支节点、叶子节点，根节点只有一个，分支节点数量不固定由oracle合理分配，分支节点存储数据的范围，最终的分支节点下面有叶子节点，叶子节点负责存储具体的值和rowid。比如:根节点存储大于50和小于50两个范围，两个分支节点，大于50的分支节点存储50-60、60-70、大于100等区段，每个区段都有对应的叶子节点，比如大于100的区段下面的叶子节点存储100、101、 102 和对应的rowid等具体数据，100、 101、 102的值即为表中建立索引的列（比如: num） 的值，当编写sql语句where num=101 时，就会通过检索索引表，查询到叶子节点上存储的rowid，然后根据rowid查询对应的数据，避免了对数据表的全表扫描。
   3. 索引也不是越多越好，索引适合建立在经常作为条件的列（即where后作为条件的列，包括关联条件）、以及order by的列。对于增删改性能要求特别高而查询要求不高的表就不建议建立索引，因为索引会降低增删改的效率。
   4. 语法：`create [unique] index index_name on tablename(column_list)[tablespace tablespace_name];`

![](https://cdn.nlark.com/yuque/0/2021/jpeg/22614707/1636697340689-a0b3411d-d139-4e9b-a754-eb0bafa38a9f.jpeg)
## SQL 语句优化

1. 对查询进行优化，要尽量避免全表扫描，首先应考虑在 where及order by涉及的列上建立索引。

2. 应尽量避免在where子句中对字段进行null值判断，否则将导致引擎放弃使用索引而进行全表扫描。

如:`select id from t where num is null`

   1. 最好不要给数据库留nulll，尽可能的使用not null填充数据库。（备注，描述，评论之类的可以设置为NULL, 其他的最好不要使用NULL）
   2. 不要以为null不需要空间，如：char(100)类型，在字段建立时，空间就固定了，不管是否插入值(NULL也包含在内)，都是占用100 个字符的空间的，如果是varchar这样的变长字段，null不占用空间。
   3. 可以在num上设置默认值0，确保表中num列没有null值，然后这样查询`select id from t where num = 0`

3. 应尽量避免在where子句中使用!=或<>操作符，否则引擎将放弃使用索引而进行全表扫描。

4. 应尽量避免在where子句中使用or来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描。
   1. 如:`select id from t where num=10 or Name='admin'`
   2. 可以这样查询:
```sql
select id from t where num = 10
union all
select id from t where Name =‘admin'
```

5. in和not in也要慎用，否则会导致全表扫描。如`select id from t where num in(1,2,3)`
   1. 对于连续的数值，能用between 就不要用in，如`select id from t where num between 1 and 3`
   2. 很多时候用exists代替in是一 个好的选择。
```sql
select num from a where num in(select num from b)
--可以用下面的语句代替
select num from a where exists(select 1 from b where num=a.num);
```

6. 如果在where子句中使用参数，也会导致全表扫描，因为SQL只有在运行时才会解折局部变量，但优化程序不能将访问计划的选择推迟到运行时，它必须在编译时进行选择，然而，如果在编译时建立访问计划，变量的值还是未知的，因此无法作为索引选择的输入项。如下面语句将进行全表扫描:

`select id from t where num = @num`
可以改为强制查询使用索引:
`select id from t with(index(索引名)) where num = @num`
应尽量避免在where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。
如:`select id from t where num/2 = 100`
应改为:`select id from t where num = 100*2`

7. 应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描，除非建立函数索引。

如：name以abc开头的id：`select id from t where substring (name, 1,3) =’abc'`
生成的id：`select id from t where datediff (day, createdate,' 2005-11-30' ) = 0`
应改为:
`select id from t where name like’ abc%'`
`select id from t where createdate >=’ 2005-11-30' and createdate <’ 2005-12-1’`

8. 不要在where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。

9. Update语句，如果只更改1、2个字段，不要Update全部字段，否则频繁调用会引起明显的性能消耗，同时带来大量日志。

10. 对于多张大数据量(这里几百条就算大了)的表JOIN,要先分页再JOIN,否则逻辑读会很高，性能很差。

11. 索引并不是越多越好，索引虽然可以提高相应的select的效率，但同时也降低了Insert及update的效率，因为insert 成update时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个， 若太多则应考虑一些不常使用到的列上建的索引是否有必要。

12. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。

13. 尽可能的使用varchar/nvarchar 代替char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

14. 任何地方都不要使用select * from t，用具体的字段列表代替“*”，不要返回用不到的任何字段。

15. 尽量避免向客户端返回大量数据，若数据量过大，应该考虑相应需求是否合理。

16. 表中有大字段X(例如: text 类型)，且字段X不会经常更新，以读为为主，请问您是选择拆成子表，还是继续放一起?写出您这样选择的理由

拆带来的问题：连接消耗+存储拆分空间；
不拆可能带来的问题：查询性能。
如果能容忍拆分带来的空间问题，拆的话最好和经常要查询的表的主键在物理结构上放置在一起(分区)顺序IO，减少连接消耗，最后这是一个文本列再加上一个全文索引来尽量抵消连接消耗。
如果能容忍不拆分带来的查询性能损失的话：上面的方案在某个极致条件下肯定会出现问题，那么不拆就是最好的选择。
## 数据库索引

1. 索引的作用？

索引作用：协助快速查询数据库表中数据。
为表设置索引要付出代价的：

   1. 增加了数据库的存储空间
   2. 在插入和修改数据时要花费较多的时间(因为索引也要随之变动)

2. 索引的优缺点？

创建索引可以大大提高系统的性能(优点) ：

   1. 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。
   2. 可以大大加快数据的检索速度，这也是创建索引的最主要的原因。
   3. 可以加快表和表之间的连接，实现数据的参考完整性。
   4. 可以显著减少查询中分组和排序的时间。
   5. 可以在查询的过程中，使用优化隐藏器，提高系统的性能。

增加索引也有许多不利的方面(缺点)：

   1. 创建索引和维护索引要耗费时间，时间随着数据量的增加而增加。
   2. 索引需要占物理空间，除了数据表占数据空间之外，每个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
   3. 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度。

3. 哪些列适合建立索引、哪些不适合建索引？

索引是建立在数据库表中的某些列的上面。在创建索引的时候，应该考虑在哪些列上可以创建索引，在哪些列上不能创建索引。
一般来说，应该在这些列上创建索引：

   1. 在经常需要搜索的列上，可以加快搜索的速度。
   2. 在作为主键的列上，强制该列的唯一性和组织表中数据的排列结构。
   3. 在经常用在连接的列上，这些列主要是一 些外键，可以加快连接的速度。
   4. 在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的。
   5. 在经常需要排序的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间。
   6. 在经常使用在WHERE子句中的列上面创建索引，加快条件的判断速度。

有些列不应该创建索引：

   1. 对于那些在查询中很少使用或者参考的列不应该创建索引。

这是因为，既然这些列很少使用到，因此有索引成者无索引，并不能提高查询速度，相反，由于增加了索引，反而降低了系统的维护速度和增大了空间需求。

   2. 对于那些只有很少数据值的列也不应该增加索引。

这是因为，由于这些列的取值很少，例如人事表的性别列，在查询的结果中，结果集的数据行占了表中数据行的很大比例，即需要在表中搜索的数据行的比例很大。增加索引，并不能明显加快检索速度。

   3. 对于那些定义为text,image和bit数据类型的列不应该增加索引。

这是因为，这些列的数据量要么相当大，要么取值很少。

   4. 当修改性能远远大于检索性能时，不应该创建索引。

这是因为，修改性能和检索性能是互相矛盾的。当增加索引时，会提高检索性能，但是会降低修改性能。当减少索引时，会提高修改性能，降低检索性能。因此，当修改性能远远大于检索性能时，不应该创建索引。

4. 为什么说B+比B树更适合实际应用中操作系统的文件索引和数据库索引？
   1. B+的磁盘读写代价更低

B+的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B树更小。如果把所有同一内部结点的关键字存放在同一盘块中， 那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。

   2. B+tree的查询效率更加稳定

由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。
# MySQL数据库

1. MySQL的MyISAM与InnoDB两种存储引擎在，事务、锁级别，各自的适用场景？
   1. 事务处理方面

MyISAM：强调的是性能，每次查询具有原子性，其执行数度比InnoDB类型更快，但是不提供事务支持。
InnoDB：支持事务，外部键等高级数据库功能。具有事务(omit)、回滚(ollback)和崩溃修复能力(crash recovery ceapabilities) 的事务安全(transaction safe (ACIDcompl iant))型表。

   2. 锁级别

MyISAM：只支持表级锁，用户在操作MyISAM表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。

2. mysql都有什么锁，死锁判定原理和具体场景，死锁怎么解决？
   1. MySQL有三种锁的级别：页级、表级、行级。

表级锁：开销小，加锁快，不会出现死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低。
行级锁：开销大，加锁慢，会出现死锁，锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
页面锁：开销和加锁时间界于表锁和行锁之间，会出现死锁，锁定粒度界于表锁和行锁之间，并发度一般

   2. 什么情况下会造成死锁?什么是死锁?

死锁：是指两个或两个以上的进程在执行过程中。因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。
表级锁不会产生死锁。所以解决死锁主要还是针对于最常用的InnoDB.死锁的关键在于:两个(或以上)的Session加锁的顺序不一致。那么对应的解决死锁问题的关键就是：让不同的session加锁有次序。

   3. 死锁的解决办法?
      1. 查出的线程杀死kill

SELECT trx_ MySQL_ thread_ id FROM information schema. INNODB _TRX;

      2. 设置锁的超时时间

Innodb行锁的等待时间，单位秒。可在会话级别设置，RDS实例该参数的默认值为50(秒)。
生产环境不推荐使用过大的innodb_ lock_ wait_ timeout参数值
该参数支持在会话级别修改，方便应用在会话级别单独设置某些特殊操作的行锁等待超时时间，如下: set innodb lock_ wait_ 
timeout1000: -设置当前会话Innodb 行锁等待超时时间，单位秒。

      3. 指定获取锁的顺序

3. mysql 高并发环境解决方案?

MySQL高并发环境解决方案：分库，分表，分布式，增加二级缓存。。。。。
需求分析：互联网单位每天大量数据读取，写入，并发性高。
现有解决方式：水平分库分表由单点分布到多点数据库中，从而降低单点数据库压力。
集群方案：解决DB宕机带来的单点DB不能访问问题。
读写分离策略：极大限度提高了应用中Read 数据的速度和并发量。无法解决高写入压力。
# 悲观锁、乐观锁
单机应用乐观锁悲观锁，select时怎么加排它锁?

1. 悲观锁(Pessimistic Lock) 

悲观锁特点：先获取锁，再进行业务操作。
悲观的认为获取锁是非常有可能失败的，因此要先确保获取锁成功再进行业务操作。通常所说的“一锁二查三更新”指的是使用悲观锁。悲观锁需要数据库本身提供支持，通过常用的select ... for update操作来实现悲观锁。当数据库执行select for update时会获取被select中的数据行的行锁，因此其他并发执行的select for update如果试图选中同一行则会发生排斥(需要等待行锁被释放)，因此达到锁的效果。select for update获取的行锁会在当前事务结束时自动释放，因此必须在事务中使用。
补充:
不同的数据库对select for update的实现和支持都是有所区别的，oracle支持select for update nowait,表示如果拿不到锁立刻报错，而不是等待，MySQL就没有no wait这个选项。
MySQL还有个问题是select for update语句执行中所有扫描过的行都会被锁上，这一点很容易造成问题。因此如果在MySQL中用悲观锁务必要确定走了索引，而不是全表扫描。

2. 乐观锁(0ptimistic Lock) 

乐观的认为多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，那么当前正在提交的事务会进行回滚。
乐观锁的特点先进行业务操作，不到万不得已不去拿锁。乐观的认为拿锁多半是会成功的，因此在进行完业务操作，需要实际更新数据的最后一步再去拿锁。乐观锁在数据库上的实现完全是逻辑的，不需要数据库提供特殊的支持。
一般的做法是在需要锁的数据上增加个版本号， 或者时间戳，

3. 实现方式举例如下:

乐观锁(给表加一个版本号字段)，这个并不是乐观锁的定义，给表加版本号，是数据库实现乐观锁的一种方式。
`SELECT data AS old data, version AS old. version FROM ...`
根据获取的数据进行业务操作，得到new_data 和new _version
```
UPDATE SET data = new data, version = new. _version WHERE version = old version
if (updated row > 0) {
//乐观锁获取成功，操作完成
}else{
//乐观锁获取失败，回滚并重试
}
```
注意:
乐观锁在不发生取锁失败的情况下开销比悲观锁小，但是一旦发生失败回滚开销则比较大，因此适合用在取锁失败概率比较小的场景，可以提升系统并发性能
乐观锁还适用于一些比较特殊的场景，例如在业务操作过程中无法和数据库保持连接等悲观锁无法适用的地方。

4. 总结:

悲观锁和乐观锁是数据库用来保证数据并发安全防止更新丢失的两种方法，例子在select .... for update前加个事务就可以防止更新丢失。悲观锁和乐观锁大部分场景下差异不大，一些独特场景下有一些差别，一般我们可以从如下几个方面来判断。
响应速度：如果需要非常高的响应速度，建议采用乐观锁方案，成功就执行，不成功就失败，不需要等待其他并发去释放锁。
冲突频率：如果冲突频率非常高，建议采用悲观锁，保证成功率，如果冲突频率大，乐观锁会需要多次重试才能成功，代价比较大。
重试代价：如果重试代价大，建议采用悲观锁。
# 分布式锁

1. 基于数据库实现分布式锁：也分为乐观锁和悲观锁(悲观锁也叫排他锁)
   1. 分布式乐观锁实现方式

在数据库表中引入一个版本号(version) 字段来实现的。
当我们要从数据库中读取数据的时候，同时把这个version字段也读出来，如果要对读出来的数据进行更新后写回数据库，则需要将version加1，同时将新的数据与新的version更新到数据表中，且必须在更新的时候同时检查目前数据库里version值是不是之前的那个version，如果是，则正常更新。如果不是，则更新失败，说明在这个过程中有其它的进程去更新过数据了。
场景:假设同一个账户，用户A和用户B都要去进行取款操作，账户的原始余额是2000，用户A要去取1500，用户B要去取1000,如果没有锁机制的话，在并发的情况下，可能会出现余额同时被扣1500和1000，导致最终余额的不正确甚至是负数。但如果这里用到乐观锁机制，当两个用户去数据库中读取余额的时候，除了读取到2000余额以外，还读取了当前的版本号version=1, 等用户A或用户B去修改数据库余额的时候，无论谁先操作，都会将版本号加1，即version=2，那么另外一个用户去更新的时候就发现版本号不对，已经变成2了，不是当初读出来时候的1,那么本次更新失败，就得重新去读取最新的数据库余额。
使用乐观锁满足的条件：锁服务要有递增的版本号version,每次更新数据的时候都必须先判断版本号对不对，然后再写入新的版本号。

   2. 分布式悲观锁实现方式

基于for update来实现加锁
当加上排它锁之后，其它线程是无法操作这条记录的。那么，这样的话，我们就可以认为获得了排它锁的这个线程是拥有了分布式锁，然后就可以执行我们想要做的业务逻辑，当逻辑完成之后，再调用上述释放锁的语句即可。

2. 基于redis缓存实现分布式锁
   1. 主要是依赖redis自身的原子操作。
   2. set user_key user_value NX PX100：redis从2.6.12版本开始，set 命令才支持这些参数

nx :只在在键不存在时，才对键进行设置操作，SET key value NX效果等同于SETNX key value
px毫秒：设置键的过期时间为millisecond毫秒，当超过这个时间后，设置的键会自动失效。
这段命令理解：当redis中不存在user key的时候，才会去设置一一个user_ key, 并且这个键的值为user.value,且这个键的寻获时间为100ms，这个命令是只有在某个key不存在的时候，才会执行成功。那么当多个进程同时并发的去设置同一个key的时候，就永远只会有一个进程成功。当某个进程设置成功之后，就可以去执行业务逻辑了，等业务逻辑执行完毕之后，再去进行解锁。解锁很简单，只需要删除这个key就可以了，不过删除之前需要判断，这个key对应的value是当初自己设置的那个。

   3. redis集群模式的分布式锁， 可以采用redis的Redlock机制。

3. 基于Zookeeper实现分布式锁

使用Zookeeper的临时有序节点来实现的分布式锁。
原理：当某客户端要进行逻辑的加锁时，就在zookeeper上的某个指定节点的目录下，去生成一个唯一的临时有序节点，然后判断自己是否是这些有序节点中序号最小的一个，如果是，则算是获取了锁。如果不是，则说明没有获取到锁，那么就需要在序列中找到比自己小的那个节点，并对其调用exist()方法，对其注册事件监听，当监听到这个节点被删除了，那就再去判断次自己当初创建的节点是否变成了序列中最小的。如果是，则获取锁，如果不是，则重复上述步骤。当释放锁的时候，只需将这个临时节点删除即可。
# JVM内存原理
由于Java程序是交由JVM执行的，所以我们在谈Java内存区域划分的时候事实上是指JVM内存区域划分。在讨论JVM内存区域划分之前，先来看一下 Java程序具体执行的过程：
![](https://cdn.nlark.com/yuque/0/2021/jpeg/22614707/1636952426965-c7717071-754f-4ece-9e51-dc1658d2b16c.jpeg)


如上图所示，首先Java源代码文件(Java 后缀)会被Java编译器编译为字节码文件(.class后缀），然后由JVM中的类加载器加载各个类的字节码文件，加载完毕后，交由JVM执行引擎执行。在整个程序执行过程中，JVM会用一段空间存储程序执行期间需要用到的数据和相关信息，这段空间般被称作为Runtime Date Ares (运行时数据区)，也就是我们常说的JVM内存。因此，在Java中我们常常说到的内存管理就是针对这段空间进行管理(如何分配和回收内存空间)。
在知道了JVM内存是什么东西之后，下面我们就来讨论下这段空间具体是如何划分区域的?
根据(Java虚拟机规范》的规定，运行时数据区通常包括这几个部分：程序计数器(Programn Counter Register)、Java 栈(VM Stack)、本地方法栈(Native Method Stack)、方法区(Method Area)、堆(Heap)。
[

](http://www.cnblogs.com/dolphin05201)

1. 程序计数器

程序计数器(Program Counter Register)，用来指示执行哪条指令的。在JVM中，多线程是通过线程轮流切换来获得CPU执行时间的，因此，在任一具体时刻，一个CPU的内核只会执行一条线程中的指令，为了能够使得每个线程都在线程切换后能够恢复在切换之前的程序执行位置，每个线程都需要有自己独立的程序计数器，并且不能互相被干扰，否则就会影响到程序的正常执行次序。因此，可以这么说，程序计数器是每个线程所私有的。
由于程序计数器中存储的数据所占空间的大小不会随程序的执行而发生改变，因此，对于程序计数器是不会发生内存溢出现象(OutOfMemory)的。

2. java栈

Java栈中存放的是一个个的栈帧，每个栈帧对应个被调用的方法、值、对象， Java栈是为执行Java方法服务的。

3. 本地方法栈

本地方法栈与Java栈的作用和原理非常相似。区别只不过是Java栈是为执行Java方法服务的，而本地方法栈则是为执行本地方法( Native Method)服务的。

4. 堆

Java中的堆是用来存储对象本身的以及数组(当然，数组引用是存放在Java栈中的)。在Java中，程序员基本不用去关心空间释放的问题，Java的垃圾回收机制会自动进行处理。因此这部分空间也是Java垃圾收集器管理的主要区域。另外，堆是被所有线程共享的，在JVM中只有一个堆。

5. 方法区

方法区在JVM中也是一个非常重要的区域，它与堆一样， 是被线程共享的区域。在方法区中，存储了每个类的信息(包括类的名称、方法信息、字段信息)、静态变量、常量以及编译器编译后的代码等。
在方法区中有一个非常重要的部分就是运行时常量池，它是每一个类或接口的常量池的运行时表示形式，在类和接口被加载到JVM后，对应的运行时常量池就被创建出来。在JVM规范中，没有强制要求方法区必须实现垃圾回收。很多人习惯将方法区称为“永久代”。
## GC (垃圾回收)机制

1. 哪些内存需要回收?

我们知道，内存运行时JVM会有一个运行时数据区来管理内存。它主要包括5大部分：程序计数器(Program Counter Register)、 虚拟机栈(VM Stack)、本地方法栈(Native Method Stack)、方法区(Method Area) 、堆(Heap) 。
而其中程序计数器、虚拟机栈、本地方法栈是每个线程私有的内存空间，随线程而生，随线程而亡，同时，java 一般内存申请有两种:静态内存和动态内存。很容易理解，编译时就能够确定的内存就是静态内存，即内存是固定的，系统一次性分配， 比如int类型变量：动态内存分配就是在程序执行时才知道要分配的存储空间大小，比如java对象的内存空间。例如栈中每一个栈帧中分配多少内存基本上在类结构定义下来时就已知了，因此这3个区域的内存分配和回收都是确定的，无需考虑内存回收的问题。
但方法区和堆就不同了，一个接口的多个实现类需要的内存可能不一样， 我们只有在程序运行期间才会知道会创建哪些对象，这部分内存的分配和回收都是动态的，GC主要关注的是这部分内存。

2. 什么时候回收?

在Java语言中，比较主流的判定一个对象已死的方法是：可达性分析(eachobii Analysis).
所有生成的对象都是个称为"GC Roots"的根的子树。从GC Roots开始向下搜索。搜索所经过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何达时，就称这个对象是不可达的(不可引用的)，也就是可以被GC回收了。

3、如何回收?
一般回收算法也有如下几种:
标记-清除(Mark-sweep)
标记所有需要回收的对象，然后统一回收。这是最基础的算法，后续的收集算法都是站于这个算法扩展的，
不足：效率低，标记清除之后会产生大量碎片。
复制(Copying)
把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一 个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现“碎片”问题。当然，此算法的缺点也是很明显的，就是需要两倍内存空间。
分代收集算法
在java程序运行的过程中，会产生大量的对象，因每个对象所能承担的职责不同所具有的功能不同所以也有着不一样的生命周期，有的对象生命周期较长，比如Http请求中的Session对象，线程，Socket 连接等：有的对象生命周期较短，比如String对象，由于其不变类的特性，有的在使用一次后即可回收。
可以将对象按其生命周期的不同划分成：年轻代(Young Generation)、年老代(old Generation)、持久代(Permanent Generation)。 其中持久代主要存放的是类信息，所以与java对象的回收关系不大，与回收息息相关的是年轻代和年老代。
年轻代：是所有新对象产生的地方。年轻代被对象填满时，就会执行GC,经过多次GC周期后，仍然存活下来的对象会被转移到年老代内存空间。通常这是在年轻代有资格提升到年老代前通过设定年龄阈值来完成的。
年老代：在年轻代中经历了多次回收后仍然没有被清除的对象，就会被放到年老代中，他们都是生命周期较长的对象。对于年老代和永久代，就不能再采用像年轻代中那样搬移腾挪的回收算法，因为那些对于这些回收战场上的老兵来说是小儿科。通常会在老年代内存被占满时将会触发Full GC,回收整个堆内存。
持久代：用于存放静态文件，比如java类、方法等。持久代对垃圾回收没有显著的影响。
# 框架原理
(请注意框架原理并不是如何使用和配置框架，而是该框架是如何实现的，比如让你写一一个框架，你该如何写? )
## Ioc原理
IOC的基本概念是：不创建对象，但是描述创建它们的方式。在代码中不直接与对象和服务连接，但在配置文件中描述哪一个组件需要哪一项服务。容器负责将这些联系在一起。
IOC(控制反转)的实现建立在工厂模式、java 反射机制和jdk 的操作XML的DOM解析方式。
Spring的bean工厂主要实现了以下几个步骤:

1. 解析配置文件(bean. xml )
2. 使用反射机制动态加载每个class 节点中配置的类
3. 为每个class节点中配置的类实例化一个对象
4. 使用反射机制调用各个对象的seter方法，将配置文件中的属性值设置进对应的对象
5. 将这些对象放在一个存储空间(beanMap) 中
6. 使用getBean 方法从存储空间(beanMap) 中取出指定的JavaBean
## Aop原理
AOP源代码实现中包含动态代理。动态代理是在不修改原有类对象方法的源代码基础上，通过代理对象实现原有类对象方法的增强，就是拓展原有类对象的功能，比如常用的日志、事务、安全等功能。日志、事务是很多方法都需要的功能，可以将打印日志、事务控制代码公共抽象出来，然后以动态代理的方式加入需要使用的方法中，那么这些方法不编写日志和事务的代码，却可以实现日志和事务的功能。

动态代理中包含一个类和一个接口

1. InvocationHandler接口
```java
public interface Invocationandler{
	public Object invoke (Object proxy, Method method, Object] args) throws Throwable;
}
```
参数说明：
Object proxy:：指最终生成的代理实例，一般不会用到。
Method method：要调用的方法
0bject[] args：方法调用时所需要的参数
可以将InvocationHandler接口的子类想象成一个代理的最终操作类。

2. Proxy类， 此类是专门完成代理的操作类，可以通过此类为一个或多个接口动态地生成实现类，此类提供了如下的操作方法：
```java
public static Object newProxyInstance (ClassLoader loader, Class<?>[] interfaces, Invocat ionHandler h) throws IllegalArgumentException
```
参数说明:
ClassLoader loader：类加载器
Class<?>[] interfaces：得到全部的接口
InvoctionHandler h：得到InvocationHlandler接口的子类实例

以下示例代码要求能默写:
示例代码:

1. 先创建一个接口
```java
package com.ls.reflect.demo;

public interface StudentDao {
    public abstract void login();
    public abstract void regist();
}
```

2. 再创建一个接口实现类
```java
package com.ls.reflect.demo;

public class StudentDaoTmpl implements StudentDao (

    public void login() {
		System.out printin("登录");
    }
    
    public void regist() {
        System.out .printIn("注册");
    }
}
```

3. 实现InvocationHandler接口
```java
package com.ls.reflect.demo;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class MyInvocationHandler implements InvocationHandler {
    private Object target;
    public Object bind(Object target){
        //绑定一个委托对象，其实就是接口实现对象
        this.target=target;
        //返回一个代理对象
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(), this);
    }
    //这里是最关键的部分，动态代理，实现方法增强，比如:调用login方法之前需要做权限和事务处理，调用完login方法之后需要打印日志
    public Object invoke(Object proxy, Method met, Object[] arg2) throws Throwable {
        System.out. print1n("假设此处为权限检查、事务处理等代码");
        Object result=met.invoke(target, arg2);
        System.out.println("假设此处为日志打印等代码");
        return result;
    }
}
```

3. 测试代码
```java
package com.ls.reflect.demo;

import java.lang.reflect.IvocationHandler;
import java.lang.reflect.Proxy;

public class StudentDemo {
    public static void main(String[] args) {
        StudentDao sd=new StudentDaoImpl();
        sd.login();
        sd.regist();
        System.out.println("----------");
        MyInvocationHandler handler=new MyInvocationHandler();
        StudentDao proxy=(StudentDao)handler.bind(sd);
        proxy.login(); //此处采用动态代理模式调用了登陆方法，那么在调用登录方法之前会执行MyInvocationHandler类中的invoke方法中的System.out.println(“假设此处为权限检查、事务处理等代码")，调用完login方法以后会执行System. out. print1n( "假设此处为日志打印等代码")。即aop前、aop后的实现方法。
        proxy.regist();
    }
}
```
# SpringMVC执行流程

1. 用户发送请求，XX/XX.do
2. 请求通过中心控制器，找到处理器映射器HandlerMapping
3. 处理器映射器返回中心控制器一个控制器Handler
4. 中心控制器找到处理器适配器HandlerAdapter
5. 处理器适配器作用到处理器，处理器开始执行
6. 处理器执行之后， 返回ModelAndView
7. ModelAndView 最终返回到中心控制器
8. 中心控制器找到视图解析器ViewResolver,通过ModelAndView中的view,来找到相应的视图
9. 将视图返回到中心控制器
10. 中心控制器会根据返回的ModelAndView中的Model来填充视图解析器返回的View
11. 将渲染后的视图返回给客户端
# 工厂模式、三种单例模式(手写)
## 工厂模式实现

1. 创建一个接口
```java
public interface Shape {
   void draw();
}
```

2. 创建接口的实现类
```java
public class Rectangle implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}
```
```java
public class Square implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}
```
```java
public class Circle implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

3. 创建一个工厂，生成基于给定信息对应的实现类对象。
```java
public class ShapeFactory {
    
   //使用 getShape 方法获取形状类型的对象
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }        
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }
}
```

4. 使用工厂，通过传递类型信息来获取实现类对象。
```java
public class FactoryPatternDemo {
 
   public static void main(String[] args) {
      ShapeFactory shapeFactory = new ShapeFactory();
 
      //获取 Circle 的对象，并调用它的 draw 方法
      Shape shape1 = shapeFactory.getShape("CIRCLE");
 
      //调用 Circle 的 draw 方法
      shape1.draw();
 
      //获取 Rectangle 的对象，并调用它的 draw 方法
      Shape shape2 = shapeFactory.getShape("RECTANGLE");
 
      //调用 Rectangle 的 draw 方法
      shape2.draw();
 
      //获取 Square 的对象，并调用它的 draw 方法
      Shape shape3 = shapeFactory.getShape("SQUARE");
 
      //调用 Square 的 draw 方法
      shape3.draw();
   }
}
```
## 单例模式实现

1. 饿汉式（类每次使用时都会初始化静态成员变量(调用该类的其它方法时也会初始化)，浪费内存）
```java
class Singleton {
    private static final Singleton instance = new Singleton();
    private  Singleton() {}
    public static Singleton getInstance() {
    return instance;
    }
}
```

2. 懒汉式（会有线程安全问题，加上同步代码块解决线程安全问题但效率低，加上双重检锁提高访问效率。没有第一次检锁时，无论单例成员变量有没有被初始化线程都会等待。加上第一次检索，当单例成员变量被初始化后无需等待直接返回对象引用）
```java
class Singleton {
    private static Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

3. 登记式（内部类只有在被外部类调用时才会加载，延缓加载时间）
```java
class Singleton {
    private Singleton() {}
    private static class Holder{
        private static final Singleton instance = new Singleton();
    }
    public static Singleton getInstance() {
        return Holder.instance;
    }
}
```
# 数据类型题

1. String是最基本的数据类型吗?

不是，java. lang. String类是final类型的，因此不可以继承这个类、不能修改这个类。为了提高效率节省空间，我们应该用StringBuffer类。

2. String和StringBuffer的区别

String和strinBuffer,它们可以储存和操作字符串，String类是数值不可改变的字符串。而StringBuffer类的字符串是可以进行修改的。当字符数据需要改变的时候就可以使用StringBuffer也可以使用StringBuffers来动态构造字符数据。

3. 说出Array List, Vector, Linked List的存储性能和特性

ArrayList和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector由于使用了synchronized 方法(线程安全)，通常性能上较Arraylist差，而LinkedList使用双向链表实现存储，按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。

4. HashMap和Hashtable的区别。

HashMap是Hashtable的轻量级实现(非线程安全的实现)，他们都完成了Map接口。
主要区别在于:

   1. HashMap允许空(null)键值(key) ，由于非线程安全，效率上可能高于Hashtable.
   2. Hashtable 的方法是Synchronize的，而HashMap不是。
   3. 在多个线程访问Hashtable时，不需为它的方法实现同步，而HashMap需提供外同步。

5. List、Map、Set 三个接口，存取元素时，各有什么特点?

List以特定次序来持有元素， 可有重复元素。Set无法拥有 重复元素，内部排序。Map保存key value值，value 可多值。
# JSP题

1. 说出Servlet的生命周期。

Serlet被服务器实例化后，容器运行其init方法，请求到达时运行其service 方法，service方法自动派遭运行与请求对应的doGet或doPost,当服务器决定将实例销毁的时候调用其destroy方法。

2. JSP 和Servlet有哪些相同点和不同点，他们之间的联系是什么?

JSP是Servlet技术的扩展，本质上是Servlet的简易方式，更强调应用的外表表达。
Servlet和JSP最主要的不同点在于:

   1. Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML里分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为. jsp的文件。
   2. JSP侧重于视图，Servlet 主要用于控制逻辑。

3. 什么情况下调用doGet()和doPost()?

Jsp页面中的form标签里的method属性为get时调用doGet()，为post时调用doPost()

4. 如何实现servlet的单线程模式

<%@page isThreadSafe= "false" %>

5. JSP 的内置对象及方法。

request：它包含了有关浏览器请求的信息，并且提供了几个用于获取cookie, header, 和session数据的有用的方法。
response：提供了几个用于设置送回浏览器的响应的方法
out：对象提供了几个方法使你能用于向浏览器回送输出结果。
pageContext：它是用于方便存取各种范围的名字空间、servlet 相关的对象的API,并且包装了通用的servlet相关功能的方法。
session：可以存贮用户的状态信息
applicaton：有助于查找有关servlet引擎和servlet环境的信息
config：该对象用于存取servlet 实例的初始化参数。
page：表示从该页面产生的一个servlet实例
exception：获取错误信息。

6. 四种会话跟踪技术

page：是代表与一个页面相关的对象和属性。一个页面由一个编译好的JavaServlet类(可以带有任何的include指令，但是没有include动作)表示。这既包括servlet又包括被编译成servlet的JSP页面。
request：是代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件(由于forward 指令和include动作的关系)
session：是代表与用于某个Web客户机的一个用户体验相关的对象和属性。一个Web会话可以也经常会跨越多个客户机请求
application ：是代表与整个Web应用程序相关的对象和属性。这实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用城

7. Request对象的主要方法。

setAttribute (String name,Object)：设置名字为name的request参数值。
getAttribute (String name)：返回由name指定的属性值
getAttributeNames()：返回request对象所有属性的名字集合，结果是一个枚举的实例
getCookies()：返回客户端的所有Cookie对象，结果是一个Cookie数组
getCharacterEncoding()：返回请求中的字符编码方式
getContentLength()：返回请求的Body的长度
getHeader (String name)： 获得HTP协议定义的文件头信息
getHeaders(String name)：返回指定名字的request Header的所有值，结果是一个枚举的实例
getHeaderNames(:·返回所以request Header 的名字，结果是一个 枚举的实例
getInputStream():返回请求的输入流，用于获得请求中的数据
getMethod():获得客户端向服务器端传送数据的方法
getParameter(String name): 获得客户端传送给服务器端的有name指定的参数值
getParameterNames():获得客户端传送给服务器端的所有参数的名字，是一个枚举的实例
getParameterValues(String name): 获得有name指定的参数的所有值
getProtocol():获取客户端向服务器端传送数据所依据的协议名称
getQueryString():获得查询字符串
getRequestURI():获取发出请求字符串的客户端地址
getRemoteAddr():获取客户端的IP地址
getRemoteHost():获取客户端的名字
getSession([Boolean create]): 返回和请求相关Session
getServerName():获取服务器的名字
getServletPath0:获取客户端所请求的脚本文件的路径
getServerPort():获取服务器的端口号
removeAttribute (String name): 删除请求中的一个属性
# 线程题

1. 线程的基本概念、线程的基本状态以及状态之间的关系

线程指在程序执行过程中，能够执行程序代码的一个执行单位， 每个程序至少都有一个线程，也就是程序本身。Java中的线程有四种状态分别是:运行、就绪、挂起、结束。

2. 多线程有几种实现方法?同步有几种实现方法?

多线程有两种实现方法，分别是继承Thread类与实现Runnable接口
同步的实现方面有两种，分别是synchronized, wait与notif

3. 启动一个线程是用run()还是start()?

启动一个线程是调用start()方法，使线程所代表的虑拟处理机处于可运行状志，这意味着它可以由JVM调度并执行。这并不意味着线程就会立即运行。run()方法可以产生必须退出的标志来停止一个线程。

4. 线程同步的方法有哪些?

wait():使一个线程处于等待状态，并且释放所持有的对象的lock.
sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法， 调用此方法要捕捉InterruptedException异常。
notify():唤醒一个处于等待状态的线程，注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级。
notifyAll ()：唤醒所有处于等待状态的线程，注意并不是给所有唤醒线程一个对象的锁，而是让它们竞争

5. sleep()和wait()有什么区别?

sleep是线程类(Thread)的方法，导致此线程暂停执行指定时间，将执行机会给其他线程，但是监控状态依然保持，到时后会自动恢复。调用sleep不会释放对象锁。
wait是Object类的方法，对此对象调用wait方法导致本线程放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象发出notify方法(或notifyAll)后本线程才进入对象锁定池准备获得对象锁进入运行状态。
sleep()方法是使线程停止一段时间的方法。在sleep时间间隔期满后，线程不一定立即恢复执行。这是因为在那个时刻，其它线程可能正在运行而且没有被调度为放弃执行，除非(a)“醒来”的线程具有更高的优先级(b)正在运行的线程因为其它原因而阻塞。
wait(是线程交互时，如果线程对一个同步对象x发出一个wait()调用，该线程会暂停执行，被调对象进入等待状态，直到被唤醒或等待时间到。

6. 什么是同步和异步?

同步：如果数据将在线程间共享。例如:正在写的数据以后可能被另一个线程读到， 或者正在读的数据可能已经被另一个线程写过了，那么这些数据就是共享数据，必须进行同步存取。
异步：当应用程序在对象上调用了一个需要花费很长时间来执行的方法，并且不希望让程序等待方法的返回时，就应该使用异步编程，在很多情况下采用异步途径往往更有效率。

7. 说出数据连接池(线程池同理)的工作机制是什么?

J2EE服务器启动时会建立一定数量的池连接，并一直维持不少于此数目的池连接。客户端程序需要连接时，池驱动程序会返回一个未使用的池连接并将其表记为忙。如果当前没有空闲连接，池驱动程序就新建一定数量的连接，新建连接的数量有配置参数决定。当使用的池连接调用完成后，池驱动程序将此连接表记为空闲，其他调用就可以使用这个连接。

8. 简述synchronized和java.util.concurrent.locks.Lock的异同?

主要相同点：Lock能完成synchronized所实现的所有功能
主要不同点：Lock有比synchronized更精确的线程语义和更好的性能。synchronized会自动释放锁，而Lock -定要求程序员手工释放，并且必须在finally从句中释放。

9. 为什么不建议使用stop()和suspend()方法为何不推荐使用

stop()方法是不安全的。它会解除由线程获取的所有锁定，而且如果对象处于一种不连贯状态，那么其他线程能在那种状态下检查和修改它们。结果很难检查出真正的问题所在。
suspend0方法容易发生死锁。调用suspend()的时候，目标线程会停下来，但却仍然持有在这之前获得的锁定。此时，其他任何线程都不能访问锁定的资源，除非被“挂起”的线程恢复运行。对任何线程来说，如果它们想恢复目标线程，同时又试图使用任何一个锁定的资源，就会造成死锁。所以不应该使用suspend(),而应在自己的Thread类中置入一个标志，指出线程应该活动还是挂起。若标志指出线程应该挂起，便用wait()命其进入等待状态。若标志指出线程应当恢复，则用一个notify()重新启动线程。

10. Jave中的23种设计模式。

工厂模式（Factory），建造模式（Builder），工厂方法模式（Factory Method），原始模型模式（Prototype），单例模式（Singleton），门面模式（Facade），适配器模式（Adapter），桥梁模式（Bridge），合成模式（Composite），装饰模式（Decorator），享元模式（Flyweight），代理模式（Proxy），命令模式（Command），解释器模式（Interpreter），访问者模式（Visitor），迭代子模式（Iterator），调停者模式（Mediator），备忘录模式（Memento），观察者模式（Observer），状态模式（State），策略模式（Strategy），模板方法模式（Template Method），责任链模式（Chain of Responsibleity）
# Redis相关的面试题

1. 什么是缓存穿透,造成缓存穿透的原因是什么,如何解决?

程序在处理缓存时，一般是先从缓存查询，如果缓存没有这个key获取为null,则会从
DB中查询，并设置到缓存中去，按这种做法，那查询一个一定不存在的数据值，由于缓存是
不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请
求都要到数据库去查询，造成缓存穿透
解决:

   1. 最好对于每一个缓存key都有一一定的规范约束，这样在程序中对不符合规则的key的请求可以拒绝(但一般key都是通过程序自动生成的)
   2. 将可能出现的缓存key的组合方式的所有数值以hash形式存储在一个很大的bitmap中<布隆过滤器> (需 要考虑如何将这个可能出现的数据的hash值之后同步到bitmap中，eg.后端每次新增一个可能的组合就同步一次，或者穷举)，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力
   3. 常用：如果对应在 数据库中的数据都不存在，我们将此key对应的value设置为一个默认的值，比如"NULL"，并设置一个缓存的失效时间。当然这个key的时效比正常的时效要小的多
2. 缓存雪崩

指的是大量缓存集中在一段时间内失效， 发生大量的缓存穿透，所有的查询都落在数据
库上，造成了缓存雪崩
解决:

   1. 这个没有完美解决办法，但可以分析用户行为，尽量让失效时间点均匀分布，设置不同的过期时间
   2. 用加分布式锁或者分布式队列的方式保证缓存的单线程(进程)写(例如， redis的SETNX)，从而避免失效时大量的并发请求落到底层存储系统上。在加锁方法内先从缓存中再获取-次(防 止另外的线程优先 获取锁已经写入了缓存)，没有再查DB写入缓存。(当然也可以，在没有获取锁(tryLock)的线程中直轮询缓存， 至超限时)

3. 缓存击穿

指的是热点key在某个特殊的场景时间内恰好失效了，恰好有大量并发请求过来了，造成DB压力
解决办法:

   1. 与缓存雪崩的解决方法类似:用加锁或者队列的方式保证缓存的单线程(进程)写,在加锁方法内先从缓存中再获取一次，没有再查DB写入缓存
   2. 还有一种比较好用的(针对缓存雪崩与缓存击穿) :

物理上的缓存是不设置超时时间的(或者超时时间比较长)，但是在缓存的对象 上增加一个属性来标识超时时间(此时间相对小)。当获取到数据后，校验数据内部的标记时间，判定是否快超时了，如果是，异步发起一个线程 (控制好并发)去主动更新该缓存，这种方式会导致一定时间内，有些请求获取缓存会拿到过期的值，看业务是否能接受而定

4. Redis Cluster (Redis集群)相关知识
   1. redis3.0版本之前只支持单例模式，在3.0版本及以后才支持集群。
   2. redis集群采用P2P模式，是完全去中心化的，不存在中心节点或者代理节点
   3. redis集群是没有统一的入口的，客户端(client) 连接集群的时候连接集群中的任意节点(node)即可，集群内部的节点是相互通信的(PING-PONG 机制)，每个节点都是一个redis实例。
   4. 为了实现集群的高可用，即判断节点是否健康(能否正常使用)，redis-cluster有这么一个投票容错机制:如果集群中超过半数的节点投票认为某个节点挂了，那么这个节点就挂了(fail) 。这是判断节点是否挂了的方法
   5. 那么如何判断集群是否挂了呢? ->如果集群中任意一个节点挂了，而且该节点没有从节点(备份节点)，那么这个集群就挂了。这是判断集群是否挂了的方法
   6. 那么为什么任意一个节点挂了(没有从节点)这个集群就挂了呢? -> 因为集群内置了16384个slot (哈希槽)，并且把所有的物理节点映射到了这16384[0-16383]个slot上，或者说把这些slot均等的分配给了各个节点。当需要在Redis集群存放-个数据(key-value)时，redis 会先对这个key进行crc16算法，然后得到一个结果。再把这个结果对16384 进行求余，这个余数会对应[0-16383]其中一个槽，进而决定key-value存储到哪个节点中。所以一旦某个节点挂了， 该节点对应的slot就无法使用，那么就会导致集群无法正常工作。
   7. Redis集群至少需要3个节点，因为投票容错机制要求超过半数节点认为某个节点挂了该节点才是挂了，所以2个节点无法构成集群
   8. 要保证集群的高可用，需要每个节点都有从节点，也就是备份节点，所以Redis集群至少需要6台服务器
# 消息队列常见面试题

1. 什么是消息队列

消息队列中间件是分布式系统中重要的组件，主要解决应用解耦，异步消息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。目前使用较多的消息队列有ActiveMQ, RabbitMQ, Kafka, RocketMQ，ZeroMQ，MetaMQ

2. 消息队列的应用场景
   1. 异步处理

有些业务不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们

   2. 流量削峰

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量无法提取预知:如果以为了能处理这类瞬间峰值访问为标准来投入资源随时待命无疑是巨大的浪费。用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃

   3. 系统解耦

降低工程间的强依赖程度，针对异构系统进行适配。在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。通过消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口， 当应用发生变化时，可以独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束

3. 消息队列的不足
   1. 降低了系统的可用性: MQ挂了，整套系统崩溃了;
   2. 提高了复杂度:加入了MQ，如何保证消息是否重复消费，消息传递的顺序性、消息丢失了怎么办。
   3. 分布式一致性问题: A系统处理完了直接返回成功了，人都以为你这个请求就成功了:但是问题是，要是BCD三个系统那里，BD两个系统写库成功了，结果C系统写库失败了，咋整?你这数据就不一致了。
4. 消息中间件集群崩溃，如何保证百万生产数据不丢失?

把消息持久化写入到磁盘上去

5. 如何保证消息的可靠性传输(如何处理消息丢失的问题)?

RabbitMQ:
(1)生产者:方案一:开启Rabbitmq事务 (同步，不推荐) ;方案二:开启confirm模式(异步，推荐)
(2)MQ:开启Rabbitmq持久化
(3)消费者:关闭Rabbitmq的 自动ACK

Kafka:
(1)给topic 设置replication.factor 参数:这个值必须大于1,要求每个partition 必须有至少2个副本。
(2)在Kafka服务端设置min.insync.replicas 参数:这个值必须大于1,这个是要求一个leader至少感知到有至少一个follower 还跟自己保持联系，没掉队，这样才能确保leader挂了还有一个follower吧。
(3)在producer 端设置acks=all: 这个是要求每条数据，必须是写入所有replica之后，才能认为是写成功了。
(4)在producer端设置retries=MAX (无限次重试的意思) :这个是要求一旦写入失败，就无限重试，卡在这里了。

6. 如何保证消息的顺序性?

RabbitMQ:
拆分多个queue, 每个queue一个consumer,就是多一些 queue而已，确实是麻烦点，或者就一个queue，但是对应一个consumer，然后这个consumer内部用内存队列做排队，然后分开给底层不同的worker来处理。

Kafka：
一个topic，一个partition，一个consumer，内部单线程消费，写N个内存queue，然后N个线程分别消费一个内存queue即可。


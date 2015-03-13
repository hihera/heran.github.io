---
layout: post
title: 性能优化之MySQL优化(二)
description: 在进行MySQL性能优化时需要考虑的一些方面
category: 技术分享
tags: [Database,MySQL]
---
{% include JB/setup %}
#性能优化之MySQL优化(二)
##本文继续[性能优化之MySQL优化(一)](http://sunheran.com/2015/03/12/mysql-sql-performance/)
#第3章 索引优化
##3-1 如何选择合适的列建立索引

###如何选择合适的列建立索引？

> 1、在`where`从句，`group by`从句，`order by`从句中出现的列

> 2、索引字段越小越好(数据库是以页为单位进行存储，在一页中存取的数据越多，一次I/O操作获取的数据量越大，I/O的操作效率越高)

<!--break--> 

> 3、离散度大的列放到联合索引的前面


    select * from payment where staff_id = 2 and customer_id = 584;

**是index(staff_id,customer_id)好？还是index(customer_id,staff_id)好？**  
由于`customer_id`的离散度更大，所以应该使用`index(customer_id,staff_id)`

**如何判断离散度？** 

![discrete_judge](http://hihera.qiniudn.com/discrete_judge.png)

查询比较列的唯一值都有什么，唯一值越多离散度越好，它的可选择性越好。

     select 
          count(distinct customer_id),
          count(distinct staff_id)
     from payment

##3-2 索引优化SQL的方法

过多的索引不仅影响数据写入的效率，也影响数据查询的效率。  
**原因**：数据库在进行查询分析的时候，首先要选择使用哪一个索引来进行查询，如果索引越多，分析的过程越慢，这样就会要减小查询效率。

###索引的维护及优化----重复及冗余索引
**重复索引**是指相同的列以相同的顺序建立的同类型的索引，如下表中primary key 和 ID列上的索引就是重复索引。

     create table test(
		id int not null primary key,
		name varchar(10) not null,
		title varchar(50) not null,
		unique(id) -- 主键已经是唯一索引了不需要在建立唯一索引
     )engine=innodb;

**冗余索引**是指多个索引的前缀列相同，或是在联合索引中包含了主键的索引，下面这个例子中`key(name,id)`就是一个冗余索引。

    create table test(
		id int not null primary key,
		name varchar(10) not null,
		title varchar(50) not null,
		key(name,id) -- innodb特性是在每一个索引的后边都附加主键信息，这里又人为添加主键，就是冗余。
    )engine=innodb;

###索引的维护及优化----查找重复及冗余索引
**1、查看表结构**

	SHOW CREATE TABLE employees.dept_emp; -- 查看建表信息
	USE INFORMATION_SCHEMA;-- 切换数据库

	SELECT
	  A.TABLE_SCHEMA AS '数据名',
	  A.TABLE_NAME AS '表名',
	  A.INDEX_NAME AS '索引1',
	  B.INDEX_NAME AS '索引2',
	  A.COLUMN_NAME AS '重复列名'
	FROM
	  STATISTICS A
	JOIN STATISTICS B 
	ON A.TABLE_SCHEMA = B.TABLE_SCHEMA
	AND A.TABLE_NAME = B.TABLE_NAME
	AND A.SEQ_IN_INDEX = B.SEQ_IN_INDEX
	AND A.COLUMN_NAME = B.COLUMN_NAME
	WHERE
	  A.SEQ_IN_INDEX = 1
	AND A.INDEX_NAME <> B.INDEX_NAME

**2、使用pt-duplicate-key-checker工具检查重复及冗余索引**

引用三个参数，分别是数据库的用户名、密码、数据库IP

	pt-duplicate-key-checker \
       -uroot \ 
       -p '123' \
       -h 12.0.0.1

##3-3 索引维护的方法

###索引的维护及优化----删除不用索引
目前MySQL中还没有记录索引的使用情况，但是在PerconMySQL和MariaDB中可以通过`INDEX_STATISTICS`表来查看哪些索引未使用，但在MySQL中目前只能通过慢查日志配合pt-index-usage工具来进行索引使用情况的分析。

	pt-index-usage \
       -uroot -p '' \
	mysql-slow.log


#第4章 数据库结构优化
##4-1 选择合适的数据类型
###选择合适的数据类型
数据类型的选择，重点在于合适二字，如何确定选择的数据类型是否合适？  
> 1、使用可以存下你的数据的最小的数据类型

> 2、使用简单的数据类型。Int要比varchar类型在mysql处理上简单。

> 3、尽可能的使用not null 定义字段，或给出默认值。

> 4、尽量少用text类型，非用不可时最好考虑分表。

**举例说明**：

> * 使用int来存储日期时间，利用 `FROM_UNIXTIME(timestr)`来转为日期格式,用`UNIX_TIMESTAMP('2015-02-01 13:12:00')`转为int格式。

> * 使用bigint来存储IP地址，利用`INET_ATON('192.168.0.1')`，`INET_NTOA(ipaddress)`两个函数来进行转换。

如果IP地址用varchar存储需要15个字节，若用bigint存储只需要8个字节，数据量很大时节省存储空间，提升效率。

**以下4-2~4-5请[参考数据库设计那些事儿章节](http://sunheran.com/2015/01/27/database-design/)**
##4-2 表的范式化优化  
##4-3 表的反范式化优化  
##4-4 表的垂直拆分  

**垂直拆分**：把原来一个有很多列的表拆分成多个表，这解决了表的宽度问题。通常垂直拆分可以按以下原则进行：
> * 把不常用的字段单独放到一个表中。
> * 把大字段独立存放到一个表中。
> * 把经常一起使用的字段放到一起。

##4-5 表的水平拆分  
**水平拆分**：为了解决单表的数据量过大的问题，水平拆分的表每一个表的结构是完全一致的。  

如何把数据平均分到n张表中？以payment表拆分为5份为例：
> 1、对customer_id（主键）进行hash运算，使用mod(customer_id,5)取出0-4个值

> 2、针对不同的hashID把数据存到不同的表中。

**挑战**：    
> 1、跨分区表进行数据查询

> 2、统计及后台报表操作
前台考虑效率，用拆分后的表进行查询，后台用汇总表操作。

#第5章 系统配置优化
##5-1 数据库系统配置优化
###操作系统配置优化
数据库是基于操作系统的，目前大多数MySQL都是安装在Linux系统之上，所以对于操作系统的一些参数配置也会影响到MySQL的性能，下面就列出一些常用的系统配置。

**网络方面的配置**  
要修改`/etc.sysctl.conf`文件 #增加tcp支持的队列数

    net.ipv4.tcp_max_syn_backlog = 65535
减少断开连接时，加快timewait资源回收

    net.ipv4.tcp_max_tw_buckets = 8000
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_fin_timeout = 10

**打开文件数的限制**  
可以使用`ulimit -a`查看目录的各位限制，如果数据库表特别多的话，可以修改`/etc/security/limits.conf`文件，增加以下内容以修改打开文件数量的限制:

    soft nofile 65535
    hard nofile 65535

除此之外最好在MySQL服务器上关闭`iptables`,`selinux`等防火墙软件。文件分区类型也需要考虑。

##5-2 MySQL配置文件优化
###MySQL配置文件

MySQL可以通过启动时指定配置参数和使用配置文件两种方法进行配置，在大数情况下配置文件位于`/etc/my.cnf`或是`/etc/mysql/my.cnf`在windows系统配置文件可以是位于`C:/windows/my.ini`文件，MySQL查找配置文件的顺序可以通过以下方法获得：

    $ /usr/sbin/mysqld --verbose --help | grep -A 1 'Default options'

注意：如果存在多个位置存在配置文件，则后面的会覆盖前面的。

`innodb_buffer_pool_size`
非常重要的一个参数，用于配置Innodb的缓冲池，如果数据库中只有Innodb表，则推荐配置量为总内存的75%。

	SELECT
		ENGINE,
		ROUND(SUM(data_length + INDEX_length)/ 1024 / 1024,1 )AS "Total MB"
	FROM
		information_schema. TABLES
	WHERE
		TABLE_SCHEMA NOT IN(
			"information_schema",
			"PERFORMANCE_SCHEMA"
		)
	GROUP BY ENGINE

建议：

	Innodb_buffer_pool_size >=Total MB

`innodb_buffer_pool_instances`   
MySQL5.5中新增加参数，可以控制缓冲池的个数默认情况下只有一个缓冲池。
一个缓冲池可能增加阻塞频率，分成多份可以增加并发性。

`innodb log` 缓冲的大小，由于日志最长每秒钟就会刷新，所以一般不用太大。

###常用配置参数说明
`innodb_flush_log_at_trx_commit `#决定数据库多长时间变更刷新到磁盘。  
关键参数，对Innodb的IO效率影响很大。默认值为1，可以取0,1,2三个值，一般建议设为2，但如果数据安全性要求比较高则使用默认值1。
0，表示每一次提交不刷新，每一秒中变更刷新一次，1最安全，保证事务不丢失，2丢失最对有1秒钟的事务丢失。

`innodb_read_io_threads`  
`innodb_write_io_threads`   
以上两个参数决定了Innodb读写的IO进程数，默认为4，可以根据cpu的核数酌情调节。

`innodb_file_per_table`  
关键参数，控制Innodb每一个表使用独立的表空间，默认为OFF，也就是所有表都会建立在共享表空间中。  
共享表空间的I/O就是一个瓶颈，Innodb的共享表空间无法收缩。这样对每一个表进行删除时，可以马上回收表使用的磁盘空间，再加上分开了多个文件，增加了并发的读写效率。  

`innodb_stats_on_metadata`  
决定了MySQL在什么情况下会刷新innodb表的统计信息，保持优化器能正确的使用到索引。

##5-3 第三方配置工具使用
###Percon Configuration Wizard
下载网站：<https://tools.percona.com/wizard>

#第6章 服务器硬件优化
##6-1 服务器硬件优化
###如何选择CPU
**思考：是选择单核更快的CPU还是选择核数更多的CPU？**  

> 1、MySQL有一些工作只能使用到单核CPU
Replicate,SQL......

> 2、MySQL对CPU核数的支持并不是越多越快。
MySQL5.5使用的服务器不要超过32核

###服务器硬件优化
Disk IO优化  
**常用RAID级别简介**

* RAID0:也称为条带，就是把多个磁盘链接成一个硬盘使用，这个级别IO最好。  
* RAID1:也称为镜像，要求至少有两个磁盘，每组磁盘存储的数据相同。  
* RAID5：也是把多个（最少3个）硬盘合并成1个逻辑盘使用，数据读写时会建立奇偶校验信息，并且奇偶校验信息和相对应的数据分别存储于不同的磁盘上。当RAID5的一个磁盘数据发生损坏后，利用剩下的数据和相应的奇偶校验信息去恢复被损坏的数据。    
* RAID1+0：就是RAID1和RAID0的结合。同时具备两个级别的优缺点。一般建议数据库使用这个级别。 
 

**SNA和NAT是否适合数据库？**（看具体的性能进行测试）

* 1、常用于高可用解决方案
* 2、顺序读写效率很高，但是随机读写不如人意。
* 3、数据库随机读写比率很高。

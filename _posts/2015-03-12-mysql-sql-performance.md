---
layout: post
title: 性能优化之MySQL优化(一)
description: 在进行MySQL性能优化时需要考虑的一些方面
category: 技术分享
tags: [Database,MySQL]
---
{% include JB/setup %}
#性能优化之MySQL优化（一）

#第1章 数据库优化简介
##1-1 MySQL优化简介
###数据库优化的目的
- 避免出现页面访问错误
	- 由于数据库连接timeout产生页面5XX错误
	- 由于慢查询造成页面无法加载
	- 由于阻塞（服务器内部锁）造成数据无法提交
- 增加数据库的稳定性
	- 很多数据库问题都是由于低效的查询引起的	
- 优化用户体验
	- 流畅页面的访问速度
	- 良好的网站功能体验
	
<!--break-->
###可以从以下几个方面进行数据库优化
![mysql_optimize_part](http://hihera.qiniudn.com/mysql_optimize_part.jpg)

> 1、SQL及索引优化：根据需求写出一个结构良好的SQL，在表中建立一些有效的索引。索引太多不仅会影响写入的效率，而且也会影响查询效率，所以索引要适量有效。
  
> 2、数据库表结构：根据数据库范式设计出简洁明了的表结构，减少冗余，表结构要有益于数据查询。
  
> 3、系统配置：MySQL数据库是基于文件的，每查询一个表，都要打开一些文件，如果文件达到了一定的限制，文件就会无法打开，进行频繁的IO操作。   

> 4、硬件：选择更适合数据库的CPU，更快的I/O，更多的内存。数据库的数据是load到内存中，在进行查询修改，内存越大，查询性能越好。CPU不是越多越好。I/O设备很快也不能减少数据慢查询与阻塞。  

#第2章 SQL语句优化
##2-1 数据准备
准备一个MySQL数据库，创建一些可操作的数据，或者可以使用MySQL提供的`sakila`数据库，从<http://dev.mysql.com/doc/index-other.html> 直接获取sql脚本。

**需要考虑的问题**：  

* SQL及索引优化
* 如何分析SQL查询

##2-2 MySQL慢查日志的开启方式和存储格式

###如何发现有问题的SQL？
**使用MySQL慢查日志对有效率问题的SQL进行监控**  

	SHOW VARIABLES LIKE 'slow_query_log'; -- 查看慢查询日志功能是否开启
	SHOW VARIABLES LIKE 'slow%'; -- 查看慢查询日志的记录位置
	SET GLOBAL slow_query_log_file = '/home/mysql/sql_log/mysql-slow.log'; -- 指定慢查询日志存取文件位置
	SHOW VARIABLES LIKE '%log%'; -- 查询相关设置索引的功能是否开启
	SET GLOBAL log_queries_not_using_indexes = ON; -- 设置记录没有使用索引的SQL查询

	SHOW VARIABLES LIKE '%long_query_time'; -- 查看慢查询的记录时间
	SET GLOBAL long_query_time = 1 ;-- 记录超过1秒的慢查询

	SET GLOBAL slow_query_log = ON; -- 开启慢查询功能
	tail -50 /home/mysql/sql_log/mysql-slow.log -- 查看日志记录内容

**慢查日志存储内容解释**：    

    \# User@Host:root[root] @localhost [] #执行SQL的主机信息
    \# Query_time: 0.000031 Lock_time: 0.000000 Rows_sent: 0 Rows_examined: 0 #SQL的执行信息 
    SET timestamp=1402029017 #SQL执行时间     
    show tables; #SQL语句的内容
 
##2-3 MySQL慢查日志分析工具之mysqldumpslow

下载安装 mysqldumpslow
命令行输入：

	mysqldumpslow -h -- 使用参数介绍

举例：
	
    mysqldumpslow -t 3 /home/mysql/sql_log/mysql-slow.log | more #列出了top3慢的SQL。

mysqldumpslow输出

![mysqldump_output](http://hihera.qiniudn.com/mysqldump_output.png)

##2-4 MySQL慢查日志分析工具之pt-query-digest

**输出到文件**  

	pt-query-digest slow-log > slow_log.report

**输出到数据库表** 

	pt-query-digest slowlog -review \
	h=127.0.0.1,D=test,p=root,P=3306,u=root,t=query_review \
	--create-reviewtable \
	--review-history t= hostname_slow

**获取使用方式**  

    pt-query-digest --help #获取使用方式
    pt-query-digest /home/mysql/sql_log/mysql-slow.log | more #列出分析结果，比mysqldumpslow更完善

**pt-query-digest的输出结果**

![pt-query-digest_output1](http://hihera.qiniudn.com/pt-query-digest_output1.png)
![pt-query-digest_output2](http://hihera.qiniudn.com/pt-query-digest_output2.png) 

##2-5 如何通过慢查日志发现有问题的SQL

> * 查询次数多且每次查询占用时间长的SQL(通常为pt-query-digest分析的前几个查询)

> * IO大的SQL,注意pt-query-digest分析中的Rows examine项(SQL中扫描的行数越多，它的I/O消耗越大)

> * 未命中索引的SQL  
注意pt-query-digest分析中Rows examine 和 Rows Send 的对比（Rows examine扫描的行数远远大于Rows Send的行数的话，说明索引的命中率并不高）

##2-6 通过explain查询和分析SQL的执行计划

###如何分析SQL查询
**使用explain查询SQL的执行计划**  

    explain select customer_id,first_name,last_name from customer;
![explain_execute](http://hihera.qiniudn.com/explain_execute.png)

**explain返回各列的含义**    

* `table`：显示这一行的数据是关于哪张表的。
* `type`：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为：
	* `const`（常数查找，主键或唯一索引）
	* `eq_reg`（范围查找，一般是唯一索引或主键的查找）
	* `ref`（常见于连接查询中，一个表是基于某一个索引的查找）  
	* `range`（基于索引的范围查找）  
	* `index`（对索引的扫描操作）
	* `ALL`（全表扫描）
* `possible_keys`：显示可能应用在这张表中的索引。如果为空，没有可用的索引。
* `key`：实际使用的索引。如果为NULL，则没有使用索引。
* `key_len`：使用的索引的长度。在不损失精确性的情况下，长度越短越好（因为MySQL的读取是以页为单位的）。
* `ref`：显示索引的哪一列被使用了，如果可能的话，是一个常数。
* `rows`：MySQL认为必须检查的用来返回请求数据的行数。
* `extra` :需要注意的返回值。
* `Using filesort`：看到这个的时候，查询就需要优化了。MySQL需要进行
额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行。
* `Using temporary`：看到这个的时候，查询需要优化了。这里，MySQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY 上。

##2-7 Count()和Max()的优化

查询最后支付时间——优化max()函数

    select max(payment_date)from payment;
    explain select max(payment_date)from payment \G

![explain_payment_column](http://hihera.qiniudn.com/explain_payment_column.jpg)

创建索引：

    create index idx_paydate on payment(pay_date);

再次查询执行计划结果如图：
![after_optimize_explain_payment](http://hihera.qiniudn.com/after_optimize_explain_payment.png)

**覆盖索引**：完全可以通过索引的信息可以查询到我们需要的查询结果(上面建的那条索引就是覆盖索引)。

**在一条SQL中同时查出2006年和2007年电影的数量——优化count()函数**

错误的方式：

    select 
    	count(release_year= '2006' or release_year = '2007') 
    from film -- 无法分开计算2006和2007年的电影数量
    select 
    	count(*) 
    from film 
    where 
    	release_year= '2006' 
    AND release_year='2007' -- 不可能同时为2006和2007，因此上有逻辑错误。

正确的写法：

    select 
    	count(release_year= '2006' or null) as '2006年电影数量',
    	count(release_year='2007' or null) as '2007年电影数量' 
    from film;

count(某一列)与count(*)比较，前者不包含NULL，后者包含值为NULL的行。

##2-8 子查询的优化
###优化子查询
通常情况下，需要把子查询优化为join查询，但在优化时要注意关联键是否有一对多的关系，要注意重复数据。

（查询sandra出演的所有影片）

    EXPLAIN SELECT
	   title,
	   release_year,
	   length
    FROM  film
    WHERE
	  film_id IN(
		SELECT film_id FROM film_actor
		WHERE actor_id IN(
				SELECT actor_id FROM actor WHERE first_name = 'sandra'
			)
	)

##2-9 GROUP BY的优化
###优化group by 查询

    EXPLAIN SELECT
	  actor.first_name,
	  actor.last_name,
	  COUNT(*)
    FROM  sakila.film_actor
    INNER JOIN sakila.actor USING(actor_id)
    GROUP BY  film_actor.actor_id;

**优化group by 查询之后**  

    EXPLAIN SELECT
	    actor.first_name,
	    actor.last_name,
	    c.cnt
    FROM
	    sakila.film_actor
    INNER JOIN (SELECT actor_id,COUNT(*) AS cnt FROM sakila.film_actor
    GROUP BY actor_id
     ) 
    AS c USING(actor_id
    );

##2-10 Limit查询的优化
###优化limit查询
limit常用于分页处理，时常会伴随`order by`从句使用，因此大多时候会使用Filesorts这样会造成大量的IO问题。

     EXPLAIN SELECT film_id,description FROM sakila.film ORDER BY title LIMIT 50,5;

**优化步骤1：使用有索引的列或主键进行Order by操作**  

    EXPLAIN SELECT film_id,description FROM sakila.film ORDER BY film_id LIMIT 50,5;
     -- 利用主键排序，扫描行数减少，扫描行数越多，I/O操作越慢。

**优化步骤2：记录上次返回的主键，在下次查询时使用主键过滤**

    EXPLAIN SELECT film_id,description FROM sakila.film WHERE film_id > 55 
    and film_id < 60
    ORDER BY film_id LIMIT 50,5;-- 避免了数据量大时扫描过多的记录，缺点是主键是顺序增长的，或者自己建立一列自增列。

**未完继续**。。。  
##[性能优化之MySQL优化(二)](http://sunheran.com/2015/03/13/mysql-index-performance/)
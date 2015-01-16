---
layout: post
title: MySQL数据库常用知识总结
description: 介绍MySQL数据库创建表、存储过程以及定时任务等相关知识。
category: 技术分享
tags: [database,MySQL]
---
{% include JB/setup %}
# MySQL数据库常用知识总结


##一、创建表（TABLE）

    DROP TABLE
    IF EXISTS ult_njd_resource_report; -- 如果表存在先删除 

    CREATE TABLE ult_njd_resource_report(
        sampledate date,
        id INT(11)NOT NULL auto_increment PRIMARY KEY,-- 主键自动增长且不空
        city_cn VARCHAR(255)DEFAULT NOT NULL,-- 默认不空
        city_apnum INT(11)DEFAULT NULL,
        town_apnum INT(11)DEFAULT NULL
    )ENGINE = MyISAM DEFAULT CHARSET = utf8;-- 指定存储方式和字符编码

<!--break-->

###存储引擎 ENGINE
MySQL中的数据用各种不同的技术存储在文件(或者内存)中。这些技术中的每一种技术都使用不同的存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力。这些不同的技术以及配套的相关功能在MySQL中被称作存储引擎(也称作表类型)。

MySQL默认配置了许多不同的存储引擎，你可以在MySQL设置文件中设置一个默认的引擎类型（使用storage_engine 选项）或者在启动数据库服务器时在命令行后面加上以下选项：

    --default-storage -engine或--default-table-type
或者在创建表时直接指定：

    CREATE TABLE mytable (id int, title char(20)) ENGINE = INNODB

还可以改变现有表使用的存储引擎:

    ALTER TABLE mytable ENGINE = MyISAM
该修改要慎重，因为对不支持同样的索引、字段类型或表大小的一个类型进行修改时可能导致丢失数据。

MySQL5.1中的存储引擎列表如下：

    SHOW ENGINES;

<img src="http://ww4.sinaimg.cn/mw690/8e91ddc1jw1emvarvoq3hj20lx079gn9.jpg"></img>

CSV： 引用由逗号隔开的用作数据库表的文件。  
BLACKHOLE：用于临时禁止对数据库的应用程序输入。  
BDB：可替代InnoDB的事务引擎，支持COMMIT、ROLLBACK和其他事务特性。    
ndbcluster：MySQL的簇式数据库引擎，尤其适合于具有高性能查找要求的应用程序。  
MyISAM：默认的插件式存储引擎，它在Web、数据仓储和其他应用环境下经常使用。  
InnoDB：用于事务处理应用程序，包括ACID事务支持。InnoDB也支持FOREIGN KEY强制。  
Memory：将所有数据保存在RAM中，在需要快速查找引用和其他类似数据时，可快速访问。    
Archive：为大量很少引用的历史、归档、或安全审计信息的存储和检索提供了解决方案。  
Federated：能够将多个分离的MySQL服务器链接起来，从多个物理服务器创建一个逻辑数据库，适合分布式。  

##二、创建存储过程（PROCEDURE）
###1、语法格式：  
    CREATE PROCEDURE procedure_name(
        IN p_date DATE,-- 输入参数，在调用存储过程时指定
        OUT p_num INT,-- 输出参数，该值可在存储过程内部被改变，并可返回 
        INOUT p_name VARCHAR(20)-- 输入输出参数，调用时指定，并且可被改变和返回 
    )
    BEGIN
        处理内容
    END
###2、举例说明
####无参数存储过程
    DELIMITER //
    DROP PROCEDURE if EXISTS ult_pro_njd_resource_report //
    create procedure ult_pro_njd_resource_report() 
    begin
      insert into ult_njd_resource_report(sampledate,id,city_cn,city_apnum,town_apnum)
      select DATE_FORMAT(DATE_ADD(NOW(),INTERVAL -1 DAY),'%y-%m-%d') sampledate,
           coalesce(t.site_cn, '全区合计') city_cn,
           SUM(t.city_apnum),
           SUM(t.town_apnum)
      FROM mit_sites t
      WHERE t.site_cn in ('噶尔县','昌都县','城关区','林芝县','那曲县','日喀则市','乃东县')
     group by t.site_cn with rollup;
    COMMIT;
    end //
    DELIMITER;

调用：

    call ult_pro_njd_resource_report(); 

**DELIMITER**  
分割符，因为MySQL默认以";"为分隔符，若我们没有声明分割符，那么编译器会把存储过程当成SQL语句进行处理，则存储过程的编译过程会报错，所以要事先用DELIMITER关键字申明当前段分隔符`DELIMITER //`，这样MySQL才会将";"当做存储过程中的代码，不会执行这些代码，用完了之后要把分隔符还原`DELIMITER;`。 
####输入参数存储过程  

    DELIMITER //                      
    CREATE PROCEDURE find_name(IN p_name VARCHAR(20))
    BEGIN
    
        IF p_name IS NULL OR p_name = '' THEN
            SELECT * FROM table1 ;
        ELSE 
          SELECT * FROM table2 WHERE NAME LIKE p_name ;
        END IF ;
    
    END//
    DELIMITER ;

调用：

    call find_name('Lucy%');

####输出参数存储过程
    DELIMITER //
    CREATE PROCEDURE find_name2(
            IN p_name VARCHAR(20),
            OUT p_num INT)
         BEGIN 
            CASE p_name 
                 WHEN 'Lucy' THEN SELECT * FROM table1;
                 WHEN 'John' THEN SELECT * FROM table2;
                 WHEN 'Hera' THEN SELECT * FROM table3;
            ELSE        
                 SELECT * FROM table4 WHERE name LIKE p_name;
            END CASE;
         SELECT FOUND_ROWS() INTO p_num;
    END
    //
    DELIMITER ;

调用：

    call find_name('Lucy%',@p_num);
查询：

    SELECT @p_num;  

###3、控制语句类型
**IF语句**

    IF       situation=1  THEN
             command1;
    ELSEIF   situation=2  THEN
             command2;
    ELSE
             command3;
    END IF ;

**CASE语句**

    CASE  situation
            WHEN 1 THEN  command1;
            WHEN 2 THEN  command2;
            WHEN 3 THEN  command3;
            ELSE         command4;
    END CASE;

**WHILE (前置判断)**

    WHILE  situation >1 DO -- 循环有可能一次不执行
           command1;
    END WHILE;

**REPEAT (后置判断)** 

    REPEAT 
          command1;-- 循环至少会执行一次
    UNTIL situation<=1  END REPEAT;

###4、操作存储过程
查看 
 
    SHOW PROCEDURE STATUS;-- 查看所有的存储过程
    SELECT * FROM mysql.proc WHERE db = 'mobile' AND type = 'PROCEDURE'--限定数据库
    SHOW CREATE PROCEDURE find_name;-- 查看创建的具体的存储过程语句

删除

    DROP PROCEDURE find_name2;

执行

    CALL search_nam('John%');-- 根据创建的存储是否需要参数判断调用是否传参


##三、创建定时器（EVENT）
自MySQL5.1.6起，开始有事件调度器(Event Scheduler)，它用做定时执行某些特定任务（例如：删除记录、对数据进行汇总等等），来取代原先只能由操作系统的计划任务来执行的工作。

查询版本：

    select version();--小于5.1，请升级版本

###1、创建语法：

    CREATE EVENT [IF NOT EXISTS] event_name --事件名称
    ON SCHEDULE schedule -- 计划任务，schedule有两种设定计划任务的方式如下：

    /---------------------------------------------------------------------------------------------------------------------------------------------------
        AT TIMESTAMP [+ INTERVAL INTERVAL] --  时间戳，用来完成单次的计划任务，时间戳需要大于当前时间。
        | EVERY INTERVAL [STARTS TIMESTAMP] [ENDS TIMESTAMP]
        
        INTERVAL: -- 时间（单位）的数量[STARTS 时间戳] [ENDS时间戳]，用来完成重复的计划任务
        quantity {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |
                    WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |
                    DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}
    ---------------------------------------------------------------------------------------------------------------------------------------------------------/
    [ON COMPLETION [NOT] PRESERVE]
    --ON COMPLETION表示当单次计划任务执行完毕后或当重复性的计划任务执行到了ENDS阶段
    --PRESERVE的作用是使事件在执行完毕后不会被Drop掉，建议使用该参数

    [ENABLE | DISABLE]--Enable表示系统将执行这个事件。Disable表示系统不执行该事件
    [COMMENT 'comment']
    -- 注释会出现在元数据中，它存在information_schema表的COMMENT列，'comment'表示将注释内容放在单引号之间，建议使用注释

    DO sql_statement;-- 表示该event需要执行的SQL语句或存储过程，可以是复合语句


###2、举例说明：
    DELIMITER //
    DROP event if EXISTS event_njd_resource_report; // -- 事件名称
    CREATE event event_njd_resource_report
    ON SCHEDULE EVERY 1 DAY STARTS '2014-12-01 05:00:00' -- 每天凌晨5点执行
    ON COMPLETION PRESERVE
    DO BEGIN
     CALL ult_pro_njd_resource_report();-- 调用存储过程
    END //
    DELIMITER;

**常用定时时间**

2天后执行：

    ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 2 DAY 

2天后开启每天执行：

    ON SCHEDULE EVERY 1 DAY STARTS CURRENT_TIMESTAMP + INTERVAL 2 DAY

2天后结束每天执行：
    
     ON SCHEDULE EVERY 1 DAY ENDS CURRENT_TIMESTAMP + INTERVAL 2 DAY

从今天开始每天凌晨1点执行：

    on schedule every 1  day starts '2014-12-02 01:00:00';

每个月的一号凌晨1点执行：  

    on schedule every 1 month starts date_add(date_add(date_sub(curdate(),interval day(curdate())-1 day),interval 1 month),interval 1 hour)
每个季度一号的凌晨1点执行：
  
    on schedule every 1 quarter starts date_add(date_add(date(concat(year(curdate()),'-',elt(quarter(curdate()),1,4,7,10),'-',1)),interval 1 quarter),interval 1 hour)

每年1月1号凌晨1点执行：

    on schedule every 1 quarter starts date_add(date_add(date(concat(year(curdate()),'-',elt(quarter(curdate()),1,4,7,10),'-',1)),interval 1 quarter),interval 1 hour)

**常用日期时间类**

ADDTIME（date2 ,time_interval //将time_interval加到date2  
CONVERT_TZ (datetime2 ,fromTZ ,toTZ ) //转换时区   
CURRENT_DATE ( ) //当前日期   
CURRENT_TIME ( ) //当前时间   
CURRENT_TIMESTAMP ( ) //当前时间戳   
DATE (datetime ) //返回datetime的日期部分   
DATE_ADD (date2 , INTERVAL d_value d_type ) //在date2中加上日期或时间   
DATE_FORMAT (datetime ,FormatCodes ) //使用formatcodes格式显示datetime   
DATE_SUB (date2 , INTERVAL d_value d_type ) //在date2上减去一个时间   
DATEDIFF (date1 ,date2 ) //两个日期差   
DAY (date ) //返回日期的天   
DAYNAME (date ) //英文星期   
DAYOFWEEK (date ) //星期(1-7) ,1为星期天   
DAYOFYEAR (date ) //一年中的第几天   
EXTRACT (interval_name FROM date ) //从date中提取日期的指定部分   
MAKEDATE (year ,day ) //给出年及年中的第几天,生成日期串   
MAKETIME (hour ,minute ,second ) //生成时间串   
MONTHNAME (date ) //英文月份名   
NOW ( ) //当前时间   
SEC_TO_TIME (seconds ) //秒数转成时间   
STR_TO_DATE (string ,format ) //字串转成时间,以format格式显示   
TIMEDIFF (datetime1 ,datetime2 ) //两个时间差   
TIME_TO_SEC (time ) //时间转秒数]   
WEEK (date_time [,start_of_week ]) //第几周   
YEAR (datetime ) //年份   
DAYOFMONTH(datetime) //月的第几天   
HOUR(datetime) //小时   
LAST_DAY(date) //date的月的最后日期  
MICROSECOND(datetime) //微秒   
MONTH(datetime) //月  
 
###3、开启EVENT
MySQL中EVENT默认是关闭的,用下面的语句查看evevt的状态，若是OFF或者0，表示是关闭。

    show VARIABLES LIKE '%sche%';

开启evevt功能： 

    SET GLOBAL event_scheduler = 1;

查看是否开启：
    
    执行show variables like 'event_scheduler';-- event_scheduler为on或者是1

如果还没有开启，执行：

    set global event_scheduler='on';


###4、操作EVENT
开启：

    alter event event_njd_resource_report on completion preserve enable;
关闭：

    alter event test_event on completion preserve disable;
查看：

    select * from  mysql.event;
修改：
    
    alter event event_name...
删除：

    drop event [IF EXISTS] event_name

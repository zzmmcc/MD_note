---
title: mysql笔记
date: 2019-07-01 15:18:51
tags: 笔记
---

###常见数据库语句

```mysql
#如果存在数据库进行删除操作
drop database if exists '数据库名';
#创建数据库
create database' 数据库名'
#条件查询：
select * from table1 where '范围'
#插入：
insert into table1(field1,field2) values(value1,value2)
#删除：
delete from table1 where '范围'
#更新：
update table1 set field1=value1 where '范围'
#查找：
select * from table1 where field1 like ’%value1%’
#排序：
select * from table1 order by field1,field2 [desc]
#总数：
select count(*) as '别名' from table1
#求和：
select sum(field1) as '别名' from table1
#平均：
select avg(field1) as '别名' from table1
#最大：
select max(field1) as '别名' from table1
#最小：
select min(field1) as '别名' from table1
#去重: 
 select distinct field1 as '别名' from table1
#注:如果是distinct 多列 是以多列一起判断是否重复
#分组: 
 select * from table1 group by xxx 
```
####列操作
```mysql
ALTER TABLE '表名' ADD COLUMN '字段名' '类型'('长度');
#eg.
ALTER TABLE sing ADD COLUMN s_level TINYINT(1);
#删除列：
alter table '表名' drop column '列名';
#修改列名MySQL：
 alter table '表名' change '原名' '新名' int;
#修改列属性：
alter table '表' modify name varchar(22);
```
####建表操作

```mysql
CREATE TABLE sing(
s_num VARCHAR(5) NOT NULL UNIQUE COMMENT '身份证号,这儿为了方便以5位算',
s_beginDate DATE COMMENT '入班时间'
#指定存储引擎为INNODB(这个在优化的时候会学习)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

CREATE TABLE dance(
d_num VARCHAR(5) NOT NULL UNIQUE COMMENT '身份证号,这儿为了方便以5位算',
d_beginDate DATE COMMENT '入班时间',
d_level TINYINT(1) COMMENT '0初级，1中级，2高级'
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```

### DROP、DELETE、TRUNCATE三者的区别
    drop:删除数据与表结构，无法回滚
    truncate:清空数据，速度快。不可回滚，实质是删除表后重新创建表结构
    delete:逐行删除数据，有日志记录，可回滚

###case when then else end
    case :指定字段
    when:当什么条件
    then:满足上面的条件 就干什么
    else:前面的条件都不满足
    end:结束语
```mysql
select s_sum, case s_level
when 1 then '初级'
when s_level=2 then '中级'
else '高级'
end '等级'
from sing
```
###视图View
    视图就是一条查询语句执行返回的结果集，目的是为了隐藏部分数据或者减少联表查询而创建的一张表。
```mysql
create view first_view (s_id,s_name,c_id,c_name)
as select s_id,s_name,c_id,c_name 
from stu s join class c on s.s_id=c.c_id
```
####视图优缺点:
    方便操作，特别是查询；更加安全，可以隐藏数据。
    但是性能可能会变慢；更改原表结构都需要更改视图。
###触发器Trigger
```mysql
create trigger trigger_name
before delete #指定触发时机
on table_name for each row #对table_name的每一行
update table_name set #old表示删除前的行数据 如果是修改的old表示修改前 new表示修改后
s_cid=null where s_cid=old.c_id
```
    MySQL可以创建以下六种触发器：
    BEFORE INSERT,BEFORE DELETE,BEFORE UPDATE
    AFTER INSERT,AFTER DELETE,AFTER UPDATE
####触发器缺点:
    1、如果需要变动整个数据集而数据集数据量又较大时，触发器效果会非常差
    2、对于批量操作并不适合使用触发器 使用触发器实现的业务逻辑在出现问题时很难进行定位，特别是设计到多个触发器的情况 协同开发时，写业务层代码如果不清楚数据库 触发器的细节，容易搞不清到底触发了那些触发器 大量使用触发器会导致代码结构容易被打乱，阅读源码困难
    3、会占用物理内存
###存储过程
    优点:
        IO非常耗费资源，使用存储过程可以减少java程序与数据库的交互次数
    缺点:
        
```mysql
#存储过程创建
#DELIMITER修改语句结束符为$$ 因为mysql是;结尾 
#但是存储过程是多个语句集 在其中每一个sql都需要;结尾 
#如果我们不修改遇到分号就结束了就不是完整的存储过程了
DELIMITER $$
#如果存在这个存储过程就删除
DROP PROCEDURE IF EXISTS `MyProcedure01`$$
#创建存储过程
# DEFINER=`root`@`localhost`： 指定IP与用户
#PROCEDURE：存储过程关键字 
#MyProcedure01:存储过程名称      ()说明没有参数，跟java方法类似
CREATE [DEFINER=`root`@`localhost`] PROCEDURE `MyProcedure01`()
#开始 可以看成java方法的{
BEGIN
#DECLARE：定义变量关键字  定义一个int类型的变量test 相当于int test
	DECLARE test INT;
#设置test的值为1  相当于test=1
	SET test:=1;
#查询test的值  相当于System.out.println(test);
	SELECT test;
#结束 可以看成java方法的 }
    END$$
#存储过程写完了 需要把结束符号换回来
DELIMITER ;
#调用存储过程
CALL MyProcedure01();
```
####使用mybatis调用存储过程
```mysql
#编写取消订单的存储过程
DELIMITER $$
#输入参数为订单order_id 输出参数为result 1 代表存储过程成功 -1代表存储过程出现异常
CREATE PROCEDURE cancel_order(IN order_id VARCHAR(30),OUT result INT)
BEGIN 
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET result:=-1;#如果出现异常设置返回值为-1
START TRANSACTION;#开启事务
SET result:=1;#设置初始值 这里设置为1
UPDATE xmcc_order SET o_statu=1 WHERE o_id=order_id; #修改订单状态
     BEGIN 
	  DECLARE done INT DEFAULT TRUE;#定义变量done用来判断
	  DECLARE product_id VARCHAR(30); #定义变量 来接收游标的商品id
	  DECLARE quantity_1 INT; #定义变量来接收游标的商品数量
	  #定义游标 存储根据订单id查询到订单项中的商品id与数量
	  DECLARE cur CURSOR FOR SELECT p_id,quantity FROM xmcc_orderdetail WHERE o_id=order_id;
	  #当游标循环结束 设置done的值为false
	  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done:=FALSE;
	  OPEN cur;#打开游标
	  WHILE done DO #循环
	  FETCH cur INTO product_id,quantity_1;#将游标的值设置到变量中
	  IF done THEN #判断 不然会多修改一次
	  UPDATE xmcc_product SET p_stock=p_stock+quantity_1 WHERE p_id=product_id;
	  END IF;
	  END WHILE;
	  CLOSE cur;
     END ;
 IF result=1 THEN #没有出现异常就提交
 COMMIT;
 ELSE 
 ROLLBACK;
 END IF;
 END ; $$
 #测试
CALL cancel_order('20001',@result);
#查看结果
SELECT @result
```
```xml
<update id="cancelOrder" parameterType="map"  statementType="CALLABLE">
    {call cancel_order(
      <!--只需要对应传递过来的map key的名字就可以了 一般都是用map来处理 当然其他也是可以的-->
      #{order_id,mode=IN,jdbcType=VARCHAR},
      #{result,mode=OUT,jdbcType=INTEGER}
      )}
</update>
```
```java
Map<Object,Object> map = new HashMap();
map.put("order_id","20001");
map.put("result",3);//使用3来查看是否更改了值
xxxMapper.cancelOrder(map);
log.info("存储过程调用成功,结果为:{}",map.get("result"));
```
###索引
    优点: 
        1.索引大大减少了存储引擎需要扫描的数据量
    	2.索引可以在某些时候帮助减少排序的时间
    	3.索引可以把随机I/O转为顺序I/O
        一句话，大大增加了检索效率		
    缺点:
        1.会降低维护效率，因为索引是通过一定的算法，算出一个值后分类放在某个位子，那么每次更新又需要去算一次
        2.索引占物理空间，算出了数据结构，总得有个地方放吧

##Mysql优化
   - 选择正确的存储引擎(MyISAM、InnoDB、MEMORY、MERGE等，优化第一步)
        MyISAM(mysql5.5以前默认):
            1.)不支持事务、不支持外键；
            2).只支持表级锁
            3).没有事务日志，故障恢复数据较麻烦
            4).分区存放文件，平均分配IO，不用花费资源去处理事务，效率较高
        InnoDB:
            1).支持事务、支持外键
            2.支持行级锁与表级锁
            3).花费资源去处理事务，效率比不上MyISAM
            4).有事务日志，恢复数据较方便
       第一步回答:首先是存储引擎的选择，因为xxxxxx，所以如果应用中需要执行大量的SELECT查询，对事务要求又不高，那么MyISAM好的选择，有事务、行级锁、恢复数据要求一般选择InnoDB
   - 锁选择
            从粒度分:
                1)、表级锁:
                      锁住整张表，开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。
                2)、行级锁:
                      开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高
            从类型分:
                1)、读锁（共享锁）:
                        两边都可以并发读取数据，但任何事务都不可以修改数据，直到已释放所有共享锁
                2)、写锁（排他锁）
                        如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获取排他锁的事务既能读数据，又能修改数据
- 避免阻塞与死锁(优化第二步)
     1、数据库阻塞的现象：第一个连接占有资源没有释放，而第二个连接需要获取这个资源。如果第一个连接没有提交或者回滚，第二个连接会一直等待下去，直到第一个连接释放该资源为止。对于阻塞，数据库无法处理，所以对数据库操作要及时地提交或者回滚。造成的原因有:网络延迟、锁的范围选择有问题等。
    避免阻塞: 
        最直接最简单的方法就是把表的数据量变小
        当然sql语句调优、提高代码质量等都是很重要的
    2、死锁: 所谓死锁<DeadLock>: 是指两个或两个以上的进程在执行过程中,因争夺资源而造成的一种互相等待的现象(跟java线程死锁类似)，表级锁不会产生死锁.所以解决死锁主要还是针对于最常用的InnoDB
    避免死锁:
        1).首先在innodb搜索引擎中，会根据算法主动进行部分死锁的检测与释放，比如上面的例子自动释放。
        2).尽量以固定的顺序访问表和行，因为一个先锁A 在去B ，一个先锁B 在去A 就容易发生死锁
        3).大事务拆小。大事务更倾向于死锁，如果业务允许，将大事务拆小。
        4).在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁概率。(牺牲效率，不建议)
        5).为表添加合理的索引。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大（后面学习索引）
接着上面的优化回答:选择了合适的搜索引擎以后，在实际操作中，通过xxxxx，尽量去避免数据库阻塞与死锁的发生。
- 表设计(优化第三步)
..................................
```txt
面试问:请谈谈mysql优化？
建议回答:我觉得mysql优化的涉及的面太广了，我就把我知道的都说一下吧!

首先应该选择合适的存储引擎，
    一般常用的是MyISAM与INNODB，xxxx区别，所以xxxxx情况选择xxxx
    
其次，在项目对mysql并发操作中经常出现阻塞、死锁等现象，那么应该xxxxx尽量去避免

然后在数据库表设计方面:
	首先：应该了解清楚需求
    其次：对表的逻辑结构设计的时候，应该尽量采用三范式来减少冗余字段（减少内存浪费），与避免更新异常
    但是，根据业务分析也可以在适当地方采取反范式的设计，采用空间换时间来提升查询效率
    然后:表的命名应该尽量xxx，在字段选择上xxxxx，比如当是整数xxxx,当时小数xxxx，当是字符串xxxx ，日期xxxx
使用索引:
    1.应该根据业务需要，在更新较少，辨识度较高，查询较多等等xxx的列上建立索引，（数据结构，优缺点自己组织一下语言）
    2.在日常书写sql的时候，多用explain去分析执行计划, 避免索引失效（sql语句优化说得越多越好）
    3.	慢查询定位(最好能说出之前我们项目中使用druid的时候配置的慢查询)
```




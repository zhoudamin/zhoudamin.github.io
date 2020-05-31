---
title: MySQL学习笔记
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-03-16 21:40:15
categories: 数据库
tags: MySQL
---

关于MySQL学习过程中的一些总结！
<!--more-->

![](http://pic.sc.chinaz.com/files/pic/pic9/201508/apic14250.jpg)
# 基础知识
MySQL默认的端口号是3306；
MySQL中的超级用户叫 root；
创建数据库： create database ；
修改数据库： alter database ；
删除数据库： drop database ；

登陆MySQL客户端 : mysql -uroot -p16213018 -P3306 -h127.0.0.1

查看数据库 ： SHOW DATABASES；

打开test数据库 :　USE test ;

创建数据表： CREATE TABLE tb1(
```mysql
        username VARCHAR(20),
        age TINYINT UNSIGNED,
        salary FLOAT(8,2) UNSIGNED 
        );
```

查看数据表： SHOW TABLES ；

查看数据表结构： SHOW COLUMNS FROM tb1 ;       

插入记录： 	INSERT tb1 VALUES('Tom',25,6843.34);

记录查找： SELECT * FROM tb1;
空值与非空：
CREATE TABLE(20) tb2(
username VARCHAR(20) NOT NULL,
age TINYINT UNSIGNED NULL
);

查表： SHOW COLUMNS tb2;

---

# 事务

事务是数据库中一个单独的执行单元（Unit）

事务必须满足4个属性：

1. 原子性：事务必须被完整执行
2. 一致性：执行前和执行后，数据库数据必须保持一致的状态
3. 隔离性：也称为独立性，当两个或多个事务并发执行时，为了保证数据的安全性，将事务隔离，不被其他正在进行的事务看到
4. 持久性：永久性，事务完成后，对数据库的更改是永久的






---




# 创建用户表以及问题表
如果存在则先删除再创建
```mysql
DROP TABLE IF EXISTS `question`;
CREATE TABLE `question` (
`id` INT NOT NULL AUTO_INCREMENT,
`title` VARCHAR(255) NOT NULL,
`content` TEXT NULL,
`user_id` INT NOT NULL,
`created_date` DATETIME NOT NULL,
`comment_count` INT NOT NULL,
PRIMARY KEY (`id`),
INDEX `date_index` (`created_date` ASC));

DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(64) NOT NULL DEFAULT ‘’,
  `password` VARCHAR(128) NOT NULL DEFAULT ‘’ ,
  ‘salt’ VARCHAR(32)  NOT NULL DEFAULT ‘’ ,
  ‘head_url’ VARCHAR(256)  NOT NULL DEFAULT ‘’  ,
  PRIMARY KEY (`id`)
  UNIQUE KEY ‘name’ (‘name’)
) ENGINE=INNODB  DEFAULT CHARSET=utf8
```
# 查询表格
```mysql
SELECT * FROM  question;
```

## 删除数据库cache
```mysql
 DROP DATABASE CACHE;
```
## 创建数据库cache
```mysql
CREATE DATABASE CACHE;  // 创建一个数据库
```

## 创建表格
```mysql
CREATE TABLE SYS_ROLE(
  ID INTEGER,
  NAME VARCHAR(255) NOT NULL
);

CREATE TABLE SYS_ROLE_USER(
  SYS_USER_ID INTEGER,
  ROLES_ID INTEGER
);



INSERT INTO SYS_ROLE(id,NAME) VALUES(1,'ROLE_ADMIN');
INSERT INTO SYS_ROLE(id,NAME) VALUES(2,'ROLE_USER');

INSERT INTO SYS_ROLE_USER(SYS_USER_ID,ROLES_ID) VALUES(1,1);
INSERT INTO SYS_ROLE_USER(SYS_USER_ID,ROLES_ID) VALUES(2,2);
```


```mysql
DROP TABLE IF EXISTS  `city`;
CREATE TABLE `city` (
  `id` INT(10) UNSIGNED NOT NULL ,
  `province_id` INT(10) UNSIGNED  NOT NULL ,
  `city_name` VARCHAR(25) DEFAULT NULL ,
  `description` VARCHAR(25) DEFAULT NULL ,
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


INSERT INTO city(id,province_id,city_name,description) VALUES(2 ,2,'长沙','我家在长沙');
```


```mysql
CREATE DATABASE Persion; 

CREATE TABLE `persion` (
  `id` INT(10) UNSIGNED NOT NULL ,
  `persion_id` INT(10) UNSIGNED  NOT NULL ,
  `persion_name` VARCHAR(25) DEFAULT NULL ,
  `description` VARCHAR(25) DEFAULT NULL ,
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```

---

# 实验楼学习笔记

## 开发准备

```mysql
# 打开 MySQL 服务
sudo service mysql start        

#使用 root 用户登录，密码为空
mysql -u root
```



## 新建数据库

```mysql
CREATE DATABASE mysql_shiyan;

CREATE DATABASE name1;
create database name2;
CREATE database name3;
create DAtabaSE name4;
```

## 连接数据库

```mys
use mysql_shiyan
```

## 新建数据表

```mysql
CREATE TABLE employee (id int(10),name char(20),phone int(12));
```

## 插入数据

```mysql
INSERT INTO employee(id,name,phone) VALUES(01,'Tom',110110110);

INSERT INTO employee VALUES(02,'Jack',119119119);

INSERT INTO employee(id,name) VALUES(03,'Rose');
```

## 主键

```mysql
create table abc
(
id int(10) primary key,
  name char(20)
);
```



## 对数据库修改

```mysql
SHOW DATABASES;
DROP DATABASE test_01;
ALTER TABLE 表名字 DROP COLUMN 列名字;
```

## 索引

```mysql
ALTER TABLE employee ADD INDEX idx_id (id);  #在employee表的id列上建立名为idx_id的索引

CREATE INDEX idx_name ON employee (name);   #在employee表的name列上建立名为idx_name的索引

show index from employee;
```

## 视图

```mysql
CREATE VIEW v_emp(v_name,v_age,v_phone) AS SELECT name,age,phone FROM employee;
select * from v_emp;
```

## 操作命令

```mysql
mysql> SELECT VERSION(), CURRENT_DATE;     //查时间

mysql> SELECT SIN(PI()/4), (4+1)*5;    //计算

mysql> SELECT VERSION(); SELECT NOW();   //现在的时间
```

---

## MySQL操作

```mysql
mysql> SHOW DATABASES;
mysql> CREATE DATABASE test;
mysql> USE test

//创建表
mysql> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
    -> species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
    
    
mysql> SHOW TABLES;    
    //描述
mysql> DESCRIBE pet;    

//装载文本到表中
mysql> LOAD DATA INFILE '/home/shiyanlou/Desktop/pet.txt' INTO TABLE pet;

//检索表
mysql> SELECT * FROM pet;

//选择特点行
mysql> SELECT * FROM pet WHERE name = 'Bowser';

//选择条件
mysql> SELECT * FROM pet WHERE birth > '1998-1-1';

//选择特殊列
mysql> SELECT name, birth FROM pet;

//选择行与列
mysql> SELECT name, species, birth FROM pet
    -> WHERE species = 'dog' OR species = 'cat';

//计算行数
mysql> SELECT COUNT(*) FROM pet;
```

---

# MySQL笔试的4个问题

[问题来源](http://www.jianshu.com/p/715f507ed4b8)

## **问题1：如何删除mysql表中的重复数据，只保留一条记录？**

先添加唯一性字段，若有则直接删除

```mysql
ALTER TABLE city1 ADD id int default 0;
alter table city1 modify id int auto_increment primary key 

delete from city2
where id not in(
select * from (select min(id) from city2 group by name having count (name)>1 ) as a 
)
```

---

## **问题2：找出表中的出现次数最多的某条记录（或某个字段）**

```mysql
SELECT sid ,count (sid) FROM city2
group by sid
order by count(sid) DESC
limit 1
```



---

## **问题3：用一条SQL语句查询出grade表中每门课都大于80分的学生姓名**

```mysql
name  course    score 
张三     语文     81
张三     数学     75
李四     语文     76
李四     数学     90
王五     语文     81
王五     数学     100
王五     英语     90

------

select name from grade
group by name
having min(分数)>80

```



---

## **问题4：将表city1中num_person字段根据人数进行重新编码**

要求:0<=x<500,000编码为1
500,000<=x<1,000,000编码为2
1,000,000<=x编码为3

```mysql
select name,
case 
    when num_persion >=0 and num_persion <500000 then 1
    when num_persion >=500000 and num_persion <1000000 then 2
    when num_persion >=1000000 then 3 end as mark 
from city1   
```

---

# 运算符案例

```mysql
mysql -h localhost -u root -p16213018

mysql> CREATE DATABASE db1;

mysql> use db1;

mysql> CREATE TABLE tmp ( note VARCHAR(100),price INT);

mysql> INSERT INTO tmp   VALUES ("this is good" , 50);

mysql> SELECT price ,price +10,price-10,price*2,price /2,price %2 FROM tmp;
+-------+-----------+----------+---------+----------+----------+
| price | price +10 | price-10 | price*2 | price /2 | price %2 |
+-------+-----------+----------+---------+----------+----------+
|    50 |        60 |       40 |     100 |  25.0000 |        0 |
+-------+-----------+----------+---------+----------+----------+


mysql> SELECT   price ,price BETWEEN 30 AND 80 , GREATEST(price ,70,30),price IN(20,50,35) ,
		price IN(10,20,35)   FROM tmp;
+-------+-------------------------+------------------------+-----------------------+--------------------+
| price | price BETWEEN 30 AND 80 | GREATEST(price ,70,30) | price IN(20,50,35) | price IN(10,20,35) |
+-------+-------------------------+------------------------+-----------------------+--------------------+
|    50 |                       1 |                     70 |                     1 |                  0 |
+-------+-------------------------+------------------------+-----------------------+--------------------+

mysql> SELECT price ,price&2 ,price|4 ,~price FROM tmp;
+-------+---------+---------+----------------------+
| price | price&2 | price|4 | ~price               |
+-------+---------+---------+----------------------+
|    50 |       2 |      54 | 18446744073709551565 |
+-------+---------+---------+----------------------+
```





---

# 查找数据 GROUP BY 和 ORDER BY

```mysql
-- 建表
show databases;
use db1;
create table orderitems
(
o_num int not null,
o_item int not null ,
f_id char(10) NOT NULL,
quantity int NOT NULL,
item_price decimal(8,2) NOT NULL,
PRIMARY KEY (o_num , o_item)
);

-- 插入数据
Insert into orderitems (o_num,o_item , f_id , quantity , item_price)
values (30001,1,'a1',10,5.2),
(30001,2,'b2',3,7.6),
(30001,3,'bs1',5,11.2),
(30001,4,'bs2',15,9.2),
(30002,1,'b3',2,20.0),
(30003,1,'c0',100,10),
(30004,1,'o2',50,2.50),
(30005,1,'c0',5,10),
(30005,2,'b1',10,8.99),
(30005,3,'a2',10,2.2),
(30005,4,'m1',10,14.99);

-- 查看下表
select * from orderitems;

-- 查找订单价格大于100的订单号和总订单价格
select o_num ,sum(quantity*item_price) AS orderTotal          	 -- 选列
FROM orderitems                                                  -- 目标表
group by o_num                                                   -- 按订单号对数据进行分组
having sum(quantity*item_price)>=100;                            -- 条件语句

-- 用ORDER BY 显示返回
select o_num ,sum(quantity*item_price) AS orderTotal          	-- 选列
FROM orderitems                                                 -- 目标表
group by o_num                                                  -- 按订单号对数据进行分组
having sum(quantity*item_price)>=100                            -- 条件语句
order by orderTotal ;                                           -- 排序输出

select * from orderitems limit 4;    -- 查询结果的前4行

select * from orderitems limit 4,3;    -- 查询结果的第4个结果开始的(不包括第4个),3行记录

-- 查询表中的总行数
select count(*) AS order_num
from orderitems;      

-- 查询o_num 的价格的平均值
select o_num , avg(item_price)  as avg_price
from orderitems
group by o_num;

-- 查询o_num=30001 的价格的平均值
select o_num , avg(item_price)  as avg_price
from orderitems
where o_num=30001;

-- 查询价格最高的
select   max(item_price)  as max_price
from orderitems;

-- 查询价格最低的
select o_num ,  min(item_price)  as min_price
from orderitems;

-- 查询各个供应商提供的价格最低的
select o_num ,  min(item_price)  as min_price
from orderitems
group by o_num;



```





---

# 记录的插入、删除、更新

```mysql
-- 创建表
create table books
(
id int(11) NOT NUll primary key ,
name varchar(50) not null ,
authors varchar(100) not null ,
price float not null ,
pubdate year not null ,
note varchar(100) not null ,
num int(11) not null default 0
);

select * from books;

-- 插入行
insert into books  
(id , name , authors ,price , pubdate , note ,num) 
values (1,'Table of AAA' ,'Dickes',23,'1995','novel',111)	;

-- 插入行 方式二
insert into books  
values (2,'Emmat' ,'Jane lura',35,'1993','joke',22);	

select * from books;

-----------------------------------------------------------------------------
	id	name	authors	price	pubdate	note	num
	1	Table of AAA	Dickes	23	1995	novel	111
	2	Emmat	Jane lura	35	1993	joke	22
------------------------------------------------------------------------------							
insert into books  
values (3,'Story of Jane' ,'Jane Tim',40,'2001','novel',0),
(4,'Lovey Day' ,'cswd',20,'2005','novel',30),
(5,'Old Land' ,'dsfre',30,'2010','law',0),
(6,'The battle' ,'safs',33,'1999','medicine',40),
(7,'Rose Hood' ,'Richard Kale',28,'2008','cartoon',28);

select * from books;

-----------------------------------------------------------------------				
	id	name	         authors	    price	pubdate	note	num
	1	Table of AAA	Dickes	        23	   1995	    novel	111
	2	Emmat	        Jane lura	    35	   1993  	joke	22
	3	Story of Jane	Jane Tim	    40	   2001  	novel	0
	4	Lovey Day	    cswd	        20	   2005	    novel	30
	5	Old Land	    dsfre        	30	   2010  	law	     0
	6	The battle	    safs	        33	   1999	    medicine	40
	7	Rose Hood	    Richard Kale	28	   2008  	cartoon	 28
-------------------------------------------------------------------------

 UPDATE books SET price =price +5 WHERE note='novel';
 select id ,name,price , note from books where note = 'novel'; 
 
 ----------------------------------------------------------------
	id	name	       price	note
	1	Table of AAA	28	   novel
	3	Story of Jane	45	   novel
	4	Lovey Day	    25	   novel
-----------------------------------------------------------------	

-- 更改价格方法二 直接改
 UPDATE books set price=40 ,note ='joke' where name = 'emmat';
 select id ,name,price , note from books where note = 'joke';
 
 ---------------------------------------------------------
	id	name	price	note
	2	Emmat	40	joke
----------------------------------------------------------				
			
-- 条件删除行
delete from books where num=0;
select * from books;

--------------------------------------------------------
	id	name	authors	price	pubdate	note	num
	1	Table of AAA	Dickes	28	1995	novel	111
	2	Emmat	Jane lura	40	1993	joke	22
	4	Lovey Day	cswd	25	2005	novel	30
	6	The battle	safs	33	1999	medicine	40
	7	Rose Hood	Richard Kale	28	2008	cartoon	28
-------------------------------------------------------------							
				
```



---






















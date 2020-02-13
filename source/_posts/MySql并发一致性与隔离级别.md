---
title: MySql并发一致性与隔离级别
date: 2020-02-12 14:28:05
tags:
---
![](https://i.loli.net/2020/02/12/VYp1vjQgPuf6TAF.jpg)

## 前言

最近复习mysql的相关知识，看到有关数据一致性和隔离级别的时候有点记不住，所以今天在这里我根据自己的理解总结一下。

<!-- more -->

首先我来简单介绍一下常见的隔离级别有哪些？

**注：** 一下测试环境均是在 *未提交读(READ UNCOMMITTED)* 的隔离条件下执行的。
### 丢失修改

session 1|session 2
--|--
create table test (id int ,name varchar(40),age int, primary key(id) );| ~
inset into test values(0,test,0);| ~
select * from test where id=0;(result is : 0 test 0)| ~
~ | select * from test where id=0;(result is : 0 test 0)
begin;|~
~ | begin;
update test set age=10 where id=0;|~
~|update test set age=20 where id=0;
select * from test where id=0;(result is: 0 test 20)|~

这个结果明显是不正确的，session 1修改了age的值，但是在它读取的时候却不是自己修改的值。

### 读脏数据

session 1|session 2
--|--
create table test (id int ,name varchar(40),age int, primary key(id) );| ~
inset into test values(0,test,0);| ~
select * from test where id=0;(result is : 0 test 0)| ~
~ | select * from test where id=0;(result is : 0 test 0)
begin|~
update test set age=10 where id=0;|~
~|select * from test where id=0;(result is: 0 test 10)
rollback;|~
select * from test where id=0;(result is: 0 test 0)|~

session 1修改的age的值，然后session 2又读取的他的值，但是这时候，session 1由于某种原因有回滚了事务，造成了session 2读取的数据并不正确，造成脏读。

### 不可重复读

session 1|session 2
--|--
create table test (id int ,name varchar(40),age int, primary key(id) );| ~
inset into test values(0,test,0);| ~
select * from test where id=0;(result is : 0 test 0)| ~
~ | select * from test where id=0;(result is : 0 test 0)
begin;|~
~|begin;
~|select * from test where id=0;(result is:0 test 0)
update test set age=10 where id=0;|~
~|select * from test where id=0;(result is:0 test 10)
...|...

由于session 1中间的修改，造成了了在同一个事务session 2中两次读取的结果不同，造成了不可重复读。

### 幻影读

session 1|session 2
--|--
create table test (id int ,name varchar(40),age int, primary key(id) );| ~
inset into test values(0,test,0);| ~
select * from test where id=0;(result is : 0 test 0)| ~
~ | select * from test where id=0;(result is : 0 test 0)
begin;|~
~|begin;
select count(*) from test where id< 10;(result is: 1)|~
~|insert into test values(1,'jonh',1);
select count(*) from test where id< 10;(result is: 2)|~
...|...

session 1读取某个区间的记录的数量，而session 2在这期间又插入一条记录。造成session 1第二次读取这个数量的时候结果不同，产生幻影读。

## 隔离级别

mysql数据库的隔离级别有以下几种：

#### 读未提交（READ UNCOMMITED ）

事务的修改，即使没有提交，对于其他的事务也是可见的。

#### 读提交（READ COMMITED）

一个事务只能看见已经提交的事务的修改。也就是说，某个事务的修改，在未提交之前，对其他的事务是不可见的。

#### 可重复读（REPEATRABLE READ）

保证在一个事务中对一条记录的多次读取结果都是一样的。这个也是MySQL默认的隔离级别。

#### 可串行化（SERIALIZABLE）

强制事务串行化执行，这样使多个事务之间互不干扰，不会出现并发一致性问题。*串行化：就是多个事务同时执行的结果与各个实物点单独执行的结果是一样的。*

该隔离级别需要通过加锁来实现，因为要使用加锁机制保证同一时间只有一个事务执行，也就是保证事务串行执行。

### 在各个隔离级别下对并发一致性的实现程度

~|脏读|不可重复读|幻影读
--|:--:|:--:|:--:|
未提交读|N|N|N
读提交|Y|N|N
可重复读|Y|Y|N
可串行化|Y|Y|Y

隔离级别越高，消耗的资源就越多，并发程度就越低。所以要根据实际情况选择对应的隔离级别。
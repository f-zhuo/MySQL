## 数据库(database) 

存放文件的容器，主要包括

- 数据：数据库存储的数据

- 元数据：数据库结构信息，约束信息和表头信息

- 日志记录：修改信息

- 统计记录：数据特征信息，如库大小

- 索引：快速检索拍好序的数据的一种数据结构

schema是所有表的结构，是数据库的逻辑结构

## 数据库基本操作

### DDL

```sql
-- 创建，if not exists,charset可选
create database if not exists d1 character set utf8mb4;

create database if not exists d1 charset=utf8;

-- 删除，if exists可选
drop database if exists d1;

-- 查看
show databases;

-- 使用
use d1;

-- 更改数据库字符集
alter database d1 charset=utf8;
```

*utf8与utf8mb4区别：utf8mb4可以显示emoji*

## 表的基本操作

### DDL

```sql
-- 创建
create table t1(
				id int not null auto_increment,
				name varchar(20) not null,
				gender char(1) comment '注释',
				primary key (id,name)) 
				engine=innodb default charset=utf8;

create table t2 like t1;

create table t3 as select * from t1;
```

auto_increment是一个自增序列，可以自主设置序列值，不用赋值

指定序列开始值

```sql
create table t1(
				id int not null auto_increment=10,
				name varchar(20) not null,
				gender char(1) comment ’注释‘,
				primary key (id,name)) 
				engine=innodb default charset=utf8;
```

```sql
-- 删除表
drop table t1;

-- 查看数据库有哪些表
show tables;

-- 查看表结构（字段，类型，主键，空值，默认值等）
show columns from t1;

describe t1;

-- 查看数据库d1中所有表的信息
show table status from d1;

-- \G表示将查询结果按列打印
show table status from d1 \G

-- 指定位置增加列 
alter table t1 add c1 int first;

alter table t1 add c2 varchar(20) after c1;

-- 删除列
alter table t1 drop c1;

-- 修改列名/数据类型
alter table t1 change c2 c3 varchar(20);

alter table t1 change c2 c2 char(20);

alter table t1 modify c2 char(20);

-- 修改表名，to可以省略 
alter table t1 rename to t2;
```


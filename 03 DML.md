# DML

```sql
-- 插入数据，全列插入或指定列插入
insert into t1 values(21,'David'),
					 (18,'hanmeimei');

insert into t1(c1) values(23);

insert into t1(c1,c2) select s1,s2 from t2;
/*注意s1，s2数据类型分别和c1，c2一致*/
```

## INSERT IGNORE INTO，INSERT INTO与replace into

设置了unique index/primary key时，在索引列插入重复数据时，INSERT IGNORE INTO会忽略待插入数据中数据库已存在的那部分，如果要插入的数据在数据库中已存在就跳过；如果数据库没有，就插入新的数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的；

INSERT INTO若插入了重复数据会报错；

replace into 会先删除掉该记录再插入新记录

```sql
-- 修改表记录，如果不设置where条件就会对全表记录更改
update t1 set c1=18，c2='lilei' where c3='M';

-- 删除表记录
SET SQL_SAFE_UPDATES = 0;
delete from t1 where c1=18;
```

where后面跟的是条件，起过滤作用

## delete、truncate、drop

delete根据过滤条件删除内容，不删除表定义，不释放空间；

truncate清空表，只删除内容，不删除表定义，释放空间；

drop删除整个表内容和定义，释放空间

# 导入导出

## 导出文件

*文件路径不能有中文*

```mssql
select * 
from t1 
into outfile 'E:/t1.txt' 
fields terminated by ',' 
lines terminated by '\r\n';

-- SQL导入失败可能是my.ini文件没有修改，将secure_file_priv修改成" "
set SQL_SAFE_UPDATES=1;
show global variables like '%secure_file_priv%';
```

## 导入文件

要先建表，再导入文件

```sql
create table score(
             stuId varchar(10),
    		 stuName varchar(10) not null,
    		 class varchar(2),
    		 Math decimal(5,1),
    		 English decimal(5,1));
    		 
load local data infile 'E:/Score.txt' 
into table score 
fields terminated by ',' 
lines terminated by '\r\n';
```

LOAD DATA 默认情况下是按照数据文件中列的顺序插入数据的，如果数据文件中的列与插入表中的列不一致，则需要指定列的顺序

```sql
load local data infile 'E:/Score.txt' 
into table score(stuId,
                 class,
                 stuName,
                 Math,
                 English)
fields terminated by ',' 
lines terminated by '\r\n';
```

MySQL8对比MySQL5的改进：

- 新增了一个窗口函数
- 索引可以被隐藏和显示，当对索引隐藏时，它不会被查询优化器所使用。可以使用这个特性用于性能调试，例如先隐藏一个索引，然后观察其对数据库的影响。如果数据库性能有所下降，说明这个索引是有用的，然后将其“恢复显示”即可；如果数据库性能看不出变化，说明这个索引是多余的，可以删掉
- 新增通用表表达式(Common Table Expressions/CTE)：方便复杂的查询
- 使用utf8mb4 作为MySQL的默认字符集
- 改进了对 JSON 的支持，添加了很多json函数

此笔记主要还是面向MySQL5

## RDBMS

全称Ralational DataBase Management System，即关系型数据库管理系统，主要有

- oracle
- mysql
- sql server
- sqlite：轻量级，用于手机移动平台

RDBMS=管理员（manager）+ 数据仓库（database）

### sql

Structured Query Language，不区分大小写，主要有

- DQL：数据查询语言，select
- DML：数据操作语言，增删改(curd)，改变记录，如insert,update,delete
- DCL ：数据控制语言，进行授权，权限回收和安全问题 ，如grant,revoke
- DDL ：数据定义语言，对数据库和表的结构定义，不改变记录，如create,drop
- TPL 事务处理语言 begin transaction,commit,rollback
- CCL 指针控制语言，通过控制指针完成表的操作 declare cursor

注释问题

```sql
-- 单行注释

/*这是
多行注释*/
```

## NoSQL

not only sql ，常见的非关系型数据库有

- MongoDB
- Redis



**SQL与NoSQL的区别**

- SQL的数据是结构化的(structured)，形式一致，存在关系
- NoSQL的数据是非结构化的，数据间不一定要关联，形式也不需要一致

## 连接MySQL

```linux
mysql -u username -p passwd -h host
```

username：用户名

passwd ：密码

host：主机名，可以是IP地址，也可以是localhost

退出连接：exit



## 数据类型

- 数值型
  - int：整型
  - tinnyint
  - smallint
  - mediumint
  - bigint
  - float：单精度
  - double：双精度
  - decimal：精度可变的浮点数
- 字符型
  - char：定长字符串
  - varchar：变长字符串
  - text：存储大文本，最大存储64K
  - binary：二进制，字节字符串
  - varbinary
  - tinytext
  - mediumtext
  - longtext
  - blob(binary large objects)：二进制对象，数据量可变
  - enum：枚举，一个属性的多个选择，选一个，若不存在该值，插入空值
  - set：集合，多个属性集合，全选/多选
- 日期型
  - date
  - time
  - datetime
  - year
  - timestamp

```sql
create table test(color enum('black','green','red'),
                  attr set('bold','italic','underline'));

insert into test(color,attr) 
		    values('black',('bold,italic'));

select * from test;

-- 数字代表enum和set的值，从1开始；在set里，数字序列是1，2，4，8...为了表示组合，如3就是1+2
insert into test(color,attr) values(1,3);

select * from test;
```

## 约束条件

限制表中的数据，保证准确和可靠性，凡是不符合约束的数据，插入时就会失败

约束条件可以在创建表时使用， 也可以在修改表的时候添加

主要有：

- 主键(primary key)：保证数据唯一和非空
- 非空(not null)：不允许填写空值
- 唯一(unique)：不允许数据重复，但可以为空
- 默认值(default)
- 外键(foreign key)：用于表连接，包含另一个表的主键，保证从表的数据和主表相关联的字段保持一致
- 检查性约束(check)：语法不报错，但无效


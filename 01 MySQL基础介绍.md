MySQL8对比MySQL5的改进：

- 新增窗口函数
- 索引可以被隐藏和显示，当对索引隐藏时，它不会被查询优化器所使用。可以使用这个特性用于性能调试，例如先隐藏一个索引，然后观察其对数据库的影响。如果数据库性能有所下降，说明这个索引是有用的，然后将其“恢复显示”即可；如果数据库性能看不出变化，说明这个索引是多余的，可以删掉
- 新增通用表表达式（Common Table Expressions/CTE）：把子查询提取出来做为一个新表，方便复杂的查询
- 使用utf8mb4 作为MySQL的默认字符集
- 改进了对 JSON 的支持，添加了很多json函数

此笔记主要还是面向MySQL5

## RDBMS

全称Ralational DataBase Management System，即关系型数据库管理系统，主要的关系型数据库有

- Oracle
- MySQL
- SQL Server
- SQLite：轻量级，用于手机移动平台
- DB2
- access

### SQL

Structured Query Language，不区分大小写，主要有

- DQL：数据查询语言，select
- DML：数据操作语言，增删改(curd)，改变记录，如insert,update,delete
- DCL ：数据控制语言，进行授权，权限回收和安全问题 如grant,revoke
- DDL ：数据定义语言，定义/删除数据库的各个对象，如表，视图，索引等。不改变记录，如create,drop，不支持rollback
- TPL ：事务处理语言 begin transaction,commit,rollback
- CCL ：指针控制语言，通过控制指针完成表的操作 declare cursor

注释问题

```sql
-- 单行注释

/*这是
多行注释*/
```

## NoSQL

Not Only SQL ，代表非关系型数据库，常见的有

- MongoDB
- Redis
- Hbase
- cloudant 

## 关系型与非关系型数据库的区别

- 关系型数据库只支持基础的数据类型，数据是结构化的，形式一致，存在关系；非关系型数据库的数据以键值对、json、图片等形式存储，支持多种格式，是非结构化的，每行记录可以有不同的字段，没有固定的表模式（table schema），数据间不一定要关联，形式也不需要一致
- 非关系型数据库不使用SQL语言，不需要SQL解析，而且数据存储在缓存中，查询速度快；关系型数据库的数据是存储在磁盘上的，可以使用SQL支持复杂查询
- 非关系型数据库数据没有耦合性，拓展性强，关系型数据库拓展性差

* 关系型数据库是行存储，一次性可以写入一条记录，同时进行检查，数据完整（写检查）；非关系型数据库是列存储，写入效率差，数据完整性不能保证
* 关系型数据库读取数据时有冗余数据，读取速度慢；非关系型数据库读取时没有冗余数据，适合大量数据的读取
* 关系型数据库遵守ACID，强调强一致性；非关系型数据库一般都是分布式的，遵守CAP理论，强调最终一致性
* 关系型数据库有事务处理，可以执行更高安全性能要求的访问，非关系型没有

## 连接MySQL

```linux
mysql -u username -p passwd -h host
```

* username：用户名

* passwd ：密码

* host：主机名，可以是IP地址，也可以是localhost

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

*char存取速度比varchar快，但空间利用率更低。char存储英文占用1个字节，中文2个字节；varchar存储英文和中文都是两个字节*

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

- 主键（primary key）：只有一个，保证数据唯一和非空，可以由多个字段组成，但字段越少越好
- 非空（not null）：不允许填写空值
- 唯一（unique）：不允许数据重复，但可以为空
- 默认值（default）
- 外键（foreign key）：可以重复，可以为空，用于表连接，和另一个表的主键相关联，保证从表的数据和主表相关联的字段保持一致。在保证数据库完整性的前提下，尽量不使用外键，因为外键影响性能
- 检查性约束（check）：可以对插入值进行约束，但MySQL的check语法不报错，无效

## 键

关键码指的是数据元素中起标识作用的数据项，例如，信息采集系统中的登录号、身份证号和姓名等。其中能起唯一标识作用的关键码称为“主关键码”，如身份证号；反之称为“次关键码”，如姓名。通常一个数据元素只有一个主码，但可以有多个次码

数据库中的关键码（key，简称键）由一个或多个属性组成，具有唯一标识的作用；分为超键（Super Key）、候选键（Candidate Key）、主键（Primary Key）和外键（Foreign Key）

关系模式：数据项的集合，表示数据项之间的关系

*  超键：在关系模式中起惟一标识作用的集合，可以是一个或多个属性，包含候选键和主键
*  候选键：最小超键，不含有多余属性的超键
*  主键：用户选作元组标识的候选键
*  外键：关系模式R中的属性k是其他关系模式的主键，那么k在关系模式R中称为外键 

如：信息采集系统有两个表，t1包括登录号，身份证号，姓名，性别，城市，t2表包括姓名，电话，住址

则：对于t1来说，（登陆号，姓名），（登录号，城市），（身份证号，性别），（登录号）都是超键；

（登录号），（身份证号）都是候选键；

登录号或者身份证号是主键；

姓名是外键

# 多实例

mysql多实例就是在同一台服务器上启用多个mysql服务，监听不同的端口，运行多个服务进程。可以相互独立，互不影响地对外提供服务，便于节约服务器资源与后期架构扩展 

实现方法有两种：一个实例对应一个配置文件；一个配置文件（my.cnf）实现多个实例

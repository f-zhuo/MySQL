# DQL

## where

```sql
-- 查找，as后跟的是别名，as可省略
select c1,c2 as c from t1 where c3=9;

select * from t1;
```

| where应用的运算符 |   含义   |
| :---------------: | :------: |
|         =         |   等于   |
|       !=,<>       |  不等于  |
|         >         |   大于   |
|         <         |   小于   |
|       `>=`        | 大于等于 |
|        <=         | 小于等于 |

where条件的字符串是不区分大小写就可匹配的，如果想区分大小写，要加上binary关键字

```sql
select * from t1 where binary name='Jack';   
```

where后面通常还可以跟上组合条件and，or，between and，in

```sql
select c1 from t1 where c2 in (select c3 from t2);

select c1 from t1 where c2='M' and c3 between 1 and 10;
```

where语句中and优先级高于or，可用小括号改变优先级

*注意，任何值和null计算，都为null，即使是 null和null，永远返回 false*

所以，null不能用=判断，只能用is null/is not null

all/any：满足全部/任意一个条件

```sql
select * from score where score > all (select score from score where id=2 or id=1);

select * from score where score > any (select score from score where id=2 or id=1);
```

any中有null值还可以计算其他非null值，all不能有非null值

除了使用where精确查询，还可以使用like/not like和通配符进行模糊查询

### 通配符

_ ：只能匹配单个字符

%：能匹配任意个字符（null除外）

```sql
select * from t1 where c1 like 'Ja%';

select * from t1 where c1 like 'Ja__';
```

### 正则

MySQL也支持正则表达式，需要用regexp（regular expression）关键字

```sql
select * from t1 where c1 regexp 'Ja';
```

|       元字符        |          说明          |
| :-----------------: | :--------------------: |
|          .          |    匹配任意一个字符    |
|          *          |       匹配任意次       |
|          +          |      匹配至少一次      |
|         ？          |     匹配0次或者1次     |
|         {n}         |        匹配n次         |
|        {n,}         |      匹配至少n次       |
|        {n,m}        |       匹配n至m次       |
| [s1,s2,s3] /[s1-s2] |    匹配其中任意一个    |
|     [^s1,s2,s3]     | 匹配除了这些的任意一个 |
|          ^          |       文本的开始       |
|          $          |       文本的结束       |
|         \b          |   单词边界，成对使用   |

*\b是新版本的用法，老版本为[:<:]和[:>:]*

## 去重

distinct

```sql
select distinct c1,c2 from t1;
```

两个字段名都要满足去重条件

## 排序

order by 

```sql
-- 按升序输出，默认
select c1 from t1 order by c2 asc;

-- 按降序输出
select c1 from t1 order by c2 desc;

-- 按多列指定顺序输出
select c1 from t1 order by c2 desc,c3;
/*先按照c2降序排列，再按照c3升序排列，若对每个列都是降序输出，则每个列后都要跟上desc*/
```

`a in (b,c)`的结果是0（a不在集合）或1（a在集合），所以可以使用in做排序，将集合里的排在最后/最前

```sql
-- 1984年的获奖者和奖项（按奖项和获奖者排序），但化学和物理要排在最后
SELECT winner, subject
FROM nobel
WHERE yr=1984 
order by subject IN ('Physics','Chemistry'),subject,winner
```

## 聚合函数

| 函数  |     含义     |
| :---: | :----------: |
| count | 非null的个数 |
|  sum  |  非null之和  |
|  max  | 非null最大值 |
|  min  | 非null最小值 |
|  avg  | 非null平均值 |

* count(*)：对所有行计数，包括null，扫描全表
* count(column)：对非null的column列计数，当column是主键，只扫描主键；当column是非主键，扫描全表
* count(1)：和count(*)效果一样，但扫描主键

无主键时，`count(1)`比`count(*)`快；有主键时，`count(主键)`比`count(*)`快；只有一个字段时`count(*)`最快

### 分组查询

group by，select后接的字段只能是group by的字段，聚合函数除外；group by后的字段和子句不能使用别名，因为group by执行顺序在select之前

```sql
-- 按年份和月对订单总额进行汇总
SELECT year(pay_time) year,
       month(pay_time) month,
       sum(pay_amount) pay_total
FROM t1
GROUP BY year(pay_time),
         month(pay_time);
```

### with rollup

与group by连用，在分组统计的基础上以第一个字段为基准，对除最右的各字段组合进行汇总

```sql
select name,
	   gender,
	   age,
	   count(*) 
from t1 
group by name,
	     gender,
	     age 
with rollup;
```

先按照name,gender,age进行分组，然后按照name,gender汇总统计，最后按照name汇总统计

```sql
select name,
	   gender,
	   age,
	   count(*) 
from t1 
group by name with rollup;
```

先按照name进行分组，然后把所有name汇总统计。汇总统计的那一行的name字段默认为null，可以用coalesce()函数来设置

**coalesce(a,b,c,···)**：若a不为null，就返回a，否则看b，若b不为null，就返回b，否则看c···若都为null，就返回null

上面的SQL可写成

```sql
select coalesce(name,'总数'),
	   gender,
	   age,
	   count(*) 
from t1 group by name with rollup;
```

**ifnull(a,b)**：若a为null，取值b，否则取值a，相当于简化版的coalesce

```sql
select ifnull(c1,c2) from t1;
```

nullif(x,y)：若x=y，则结果为null，否则为x

## 数据类型变换

```mysql
select cast('1' as int) num;
```

## 行列互换

### 行转列

一个字段有多个可能的值，将这些值变成新字段

#### sum(case when)

若不需要分组，就不用sum

```sql
create table score
(id int,
userid varchar(5),
subject varchar(10),
score decimal(5,3),
primary key(id))
engine=innodb default charset=utf8;

insert into score values(1,'001',90,'yuwen'),(2,'001',93,'shuxue'),(3,'002',87.4,'yuwen'),(4,'002',85,'shuxue');
```

原始表如下：

![](.\pictures\DQL_example1.1.jpg)

```sql
select userid,
	   sum(case subject when 'yuwen' then score else 0 end) yuwen,
	   sum(case subject when 'shuxue' then score else 0 end) shuxue
from score
group by userid;

```

经过行转列的表：

![](.\pictures\DQL_example1.2.jpg)

对于case when，若不求和，只会得到第一次userid出现的subject的score

![](.\pictures\DQL_example1.3.jpg)

#### sum(if)

用法和case when一样，只是不能用于多条件

```sql
select userid,
	   sum(if(subject='yuwen',score, 0)) yuwen,
	   sum(if(subject='shuxue',score, 0)) shuxue 
from score group by userid;

```

### 列转行

多个字段变为某列字段的多行，初始表：

![](.\pictures\DQL_example4.1.jpg)

```sql
select userid,
	   yuwen as score,
	   'yuwen' as subject 
from score1 
union 
select userid,
	   shuxue as score,
	   'shuxue' as subject 
from score1;

```

列转行：

![](.\pictures\DQL_example4.2.jpg)

## 连接函数

### concat()

MySQL中字符串不允许用+连接，只能使用concat，若concat连接的记录中有一个字段为null，则返回null

```sql
select userid,concat(score,subject) ss from score;

```

![](.\pictures\DQL_example3.1.jpg)

### concat_ws()

指定连接分隔符，分隔符不能为null，否则返回null

```sql
select userid,concat_ws('%',score,subject) ss from score;

```

![](.\pictures\DQL_example3.2.jpg)

### group_concat()

与group by连用，把分组之后的多行记录放在一条记录的一个字段里。可以对原纪录去重，排序，指定分隔符

```sql
select userid,
	   group_concat(subject) subject 
from score 
group by userid; //直接分组连接，不指定分隔符，默认‘,’

```

![](.\pictures\DQL_example2.1.jpg)

```sql
select userid,
	   group_concat(subject separator'%') subject 
from score 
group by userid;//指定分隔符

```

![](.\pictures\DQL_example2.2.jpg)

```sql
select userid,
	   group_concat(subject,'%') subject 
from score 
group by userid; //指定多字段连接，可以是自定义的字段

```

![](.\pictures\DQL_example2.3.jpg)

```sql
select userid,
       group_concat(subject,'%',score) subject 
from score 
group by userid; //3个字段连接

```

![](.\pictures\DQL_example2.4.jpg)

```sql
select userid,
	   group_concat(subject,'%',score separator '//') subject 
from score 
group by userid; //指定分隔符的多字段连接

```

![](.\pictures\DQL_example2.5.jpg)

```sql
select userid,
       group_concat(score order by score) score 
from score 
group by userid; //默认升序

```

![](.\pictures\DQL_example2.6.jpg)

```sql
select userid,
	   group_concat(score order by score desc) score 
from score 
group by userid;

```

![](.\pictures\DQL_example2.7.jpg)

```sql
select userid,
	   group_concat(distinct score) score 
from score 
group by userid; //去重

```

## 字符串函数

### substring()

```sql
select substring('abcde',3,3);
select substring('abcde',3);

```

### length()

```sql
select length('abcde');

```

### replace()

```sql
select replace('abcde','c','d')

```

### instr()

```mysql
select instr('abcde','cd')

```

## round()

 显示欧洲的国家name和每个国家的人口population，以和德国的人口占比形式显示

```mysql
select name,
concat(round(population/(select population
from world
where name = 'Germany')*100,0),'%') population
from world
where continent = 'Europe';
// round 保留小数位数是固定的，round(xxx,2)*100是两位小数，round(xxx*100,0)是整数

```

## 日期时间函数

增加日期和时间（默认单位天）

```sql
SELECT adddate('1989-01-28',7);
SELECT adddate('1989-01-28',interval 7 month);
select adddate('1989-01-28 18:3:26',interval '3:4' minute_second);
SELECT date_add('1989-01-28',interval 7 year);
select adddate('1989-01-28 18:3:26',interval 1 second);
select date_add('1989-01-28 18:3:26',interval '1 3' day_hour);
select adddate('1989-01-28',interval 12 hour);

```

增加时间 

```sql
-- 给18:34:23增加1小时2分钟37秒
SELECT addtime('18:34:23','1:2:37');
-- 给1989年1月21日的18:34:23增加1天1小时2分钟37秒
SELECT addtime('1989-01-21 18:34:23','1 1:2:37')

```

返回当前时间和日期

```sql
SELECT curtime();
SELECT curdate();
SELECT now();

```

获取给定日期和时间

```sql
SELECT time('2018-3-12 17:12:34');
SELECT date('2018-3-12 17:12:34');

```

返回两个日期之差

```sql
SELECT datediff('2019-3-4','2017-6-24');
SELECT datediff('2019-3-4 12:34:54','2017-6-24 13:23:4');

```

格式化日期

| 格式化符号 |        含义         |
| :--------: | :-----------------: |
|     %Y     |   四位数形式的年    |
|     %y     |   两位数形式的年    |
|     %M     |       英文月        |
|     %m     |    阿拉伯数字月     |
|     %D     |     英文形式天      |
|     %d     |    阿拉伯数字天     |
|     %W     |   英文形式星期几    |
|     %w     | 数字形式星期几(0-6) |

```sql
SELECT date_format('20180923','%Y-%m-%d');
SELECT date_format('20180923','%W %M %D %Y');

```

返回具体日期的年，月，日，小时，分钟，秒，星期

```sql
SELECT year('2018-9-23 23:34:12');
SELECT hour('2018-9-23 23:34:12');
SELECT hour('2018-9-23');
SELECT dayofweek('2018-9-23'); 

```

## HAVING

对group by后的记录进行条件过滤，和WHERE区别在于，where是对分组之前进行过滤

```sql
-- 查看营销总额大于300的月
SELECT substring(pay_time,1,7) time,
       sum(pay_amount) pay_total
FROM t1
GROUP BY substring(pay_time,1,7)
HAVING sum(pay_amount)>300;

```

## limit  

`limit start_row,row_counts `

`limit row_counts offset start_row `

start_row默认是0，即第一行记录

```sql
select * from t1 limit 10; //select * from t1 limit 0,10;
select * from t1 limit 10,20; //从11到30行
select * from t1 limit 20,-1; //从21到最后一行

```

*limit后是不允许接表达式的，比如limit n-1就是非法的*

## 随机查询

随机查询出50条记录

```sql
select * from t1 order by rand() limit 50;

```

这种方法每次都会对所有数据排序，显然可以优化

```sql
select *
from t1
where id >
(select round(rand() * (select max(id)
from t1)))
order by id
limit 50;

```

## MySQL的执行顺序

from -on -join -where -group by -having -select -distinct -order by -limit

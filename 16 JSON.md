## Json

Json可以把js对象转换成字符串，提高数据的传输性，同时Json的结构更清楚，便于阅读

Json的值可以是数字，字符串，false/ture/null（必须是小写），数组和对象，其中最常用的是数组和对象

* Json数组是用[]包围的一组值，比如[1，2，3，4]

* Json对象由键值对构成，如{'name':'lilei','age':18,'gender':'M'}

* Json字符串，如'json'

* Json数字，如1

Json本质是一个**字符串**，实际上Json应该是'[1，2，3，4]'，'{'name':'lilei','age':18,'gender':'M'}'，'"json"'，'1'

MySQL也支持json数据格式，MySQL5.7.8开始，MySQL提供了一个原生的json类型，在此之前json是以字符串的形式存储的

新类型的优势

* JSON数据类型，会自动校验数据是否为JSON格式，如果不是，则会报错
* MySQL提供了一组操作JSON数据的内置函数
* 优化了存储格式，存储在JSON列中的JSON数据被转换成内部的存储格式，可以快速读取
* 可以修改特定的键值，而不需要更新整个JSON内容

### MySQL对JSON的操作

#### 创建

* 创建含json类型字段的表

```sql
CREATE TABLE t_json(
    				id INT PRIMARY KEY, 
    				name VARCHAR(20) , 
    				info  JSON);
```

* 构造一个JSON对象

JSON_OBJECT()

```sql
SELECT JSON_OBJECT('name', 'lilei', 'age', 18);
/*
{"name": "lilei", "age": 18}           
*/
```

* 构造一个JSON数组

JSON_ARRAY

```sql
SELECT JSON_ARRAY(1, "abc", null, true, CURTIME())); 
/*
[1, "abc", null, true, "10:37:08.000000"]
*/
```

#### 更新

* 插入含json类型字段的记录

```sql
INSERT INTO t_json(id,sname,info) 
VALUES( 1, 'lilei', JSON_ARRAY(1, "abc", null, true, CURTIME()));
```

* 增加/修改含json类型字段的记录

```sql
-- json_set()函数
UPDATE t_json SET info = json_set(info,'$.ip','192.168.1.1') WHERE id = 2; 

UPDATE t_json SET info = json_set(info,'$.ip','192.168.1.2') WHERE id = 2;
```

#### 删除

* 删除含json类型字段的记录

```sql
-- json_remove()函数
UPDATE t_json SET info = json_remove(info,'$.ip') WHERE id = 2;
```

#### 查询

* JSON_CONTAINS()

 查看指定数据是否包含在指定path里，包含则返回1，否则返回0。如果有参数为NULL或path不存在，则返回NULL

```sql
-- JSON_CONTAINS(json_doc, val[, path])
set @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
SELECT JSON_CONTAINS(@j, '4', '$.c.d'); 
/* 
1
*/
```

* JSON_TYPE()

```sql
-- 查看json字符串类型
select JSON_TYPE('[1,2,3]');

/* 
 ARRAY 
 */
 
select JSON_TYPE('"json"');

/*
 STRING
 */
 
select JSON_TYPE('json');

/*ERROR 3146 (22032): Invalid data type for JSON data in argument 1
to function json_type; a JSON string or JSON type is required.*/
```

* JSON_VALID()

检查是否是正确的JSON类

```sql
SELECT JSON_VALID('null'), JSON_VALID('NULL');
/*
|JSON_VALID('null') |  JSON_VALID('NULL') |
+--------------------+--------------------+-
|     	1	       |            0        |
*/
```

* JSON_MERGE()

合并两个JSON

```sql
SELECT JSON_MERGE('{"a": 1, "b": 2}', '{"c": 3, "a": 4}');
/*
{"a": [1, 4], "b": 2, "c": 3} 
*/
```

* JSON_EXTRACT()

JSON提取函数

```sql
SELECT JSON_EXTRACT('{"id": 14, "name": "lilei"}', '$.name');
/*
"lilei"
*/

SELECT '{"id": 14, "name": "lilei"}'->"$.name";
/*
"lilei"
*/

SELECT JSON_EXTRACT('['a','b','c']','$[0]');
/*
"a"
*/
```

* JSON_KEYS()

以json_array的形式返回json_object的所有键值

```sql
SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}'); 
/* 
["a", "b"]
*/
```

* JSON_SEARCH()

查询包含指定字符串的paths，并作为一个json array返回。如果有参数为NUL或path不存在，则返回NULL

```sql
/*
JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])

one_or_all："one"表示查询到一个即返回；"all"表示查询所有
search_str：要查询的字符串, 可以用LIKE里的'%'或‘_’匹配
path：在指定path下查
*/
SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
SELECT JSON_SEARCH(@j, 'one', 'abc'); -- "$[0]"
SELECT JSON_SEARCH(@j, 'all', 'abc'); -- ["$[0]", "$[2].x"]
SELECT JSON_SEARCH(@j, 'all', 'abc', NULL, '$[2]'); -- "$[2].x"
SELECT JSON_SEARCH(@j, 'all', '10'); -- "$[1][0].k"
SELECT JSON_SEARCH(@j, 'all', '%b%'); -- ["$[0]", "$[2].x", "$[3].y"]
SELECT JSON_SEARCH(@j, 'all', '%b%', NULL, '$[2]'); -- "$[2].x"
```


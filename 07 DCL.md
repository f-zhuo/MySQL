## DCL

```sql
-- 显示服务器错误和警告信息
show errors;

show warnings;

-- 查看当前时间
select now();	

-- 服务器版本信息
select version();

-- 服务器状态
show status;

-- 服务器配置变量
show varaibles;

-- 当前数据库名
select database();

-- 当前用户名
select user();

-- 返回当前客户端的连接id
select connection_id();

-- 创建新用户，test用户名，123密码

create user test@ip identified by '123';

-- root赋予普通用户权限，d1数据库

grant select,create on d1.* to test@ip;
grant all on d1.* to test@ip;

-- 移除用户权限

revoke select,create on d1.* from test@ip;

-- 查看用户权限

show grants for test@ip;

-- 删除用户

drop user test@ip;
```


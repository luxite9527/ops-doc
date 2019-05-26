# MySQL调试命令概要

- 查看当前MySQL进程。如果负载过高，可以通过它查出那些进程在等待，优化SQL

        show full processlist;

       主要看state列:
       Sleeping：正在等待客户端发送新请求。
       checking table: 正在检查数据表（自动）
       Closing table: 正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
       Connect Out: 复制从服务器正在连接主服务器
       Copying to tmp table on disk: 临时结果集大于tmp_table_size值，正在将临时表从内存存储转为磁盘存储
       Creating tmp table: 正在创建临时表
       Sending data: 正在处理SELECT查询，同时把结果发送到客户端
       Sorting for group: 正在进行GROUP BY 操作
       Sorting for order: 正在进行ORDER BY 操作
       Removing duplicates: 正在执行一个DISTING查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。
       Repair by sorting: 修复指令正在排序以创建索引

- 查看当前运行状态信息

        show status\G

- 查看存储引擎相关

        show engines;

- 查看表信息

        show table status like 'TableName'\G

- 查看某个系统变量

        show variables like "%server-id%";

- 创建新表时复制已存在的表结构与索引信息

        create table TableName like OldTableName;

- 查看表索引

        show index from TableName;

- 性能监测

        pt-query-digest
        explain
        show profiling

- 查看视图触发器信息

        select * from information_schema.VIEWS\G

- 查看视图定义

        show create view ViewName\G

- 查看数据库各表占用空间情况

```sql
 SELECT TABLE_NAME,
 DATA_LENGTH+INDEX_LENGTH,
 TABLE_ROWS,
 concat(round((DATA_LENGTH+INDEX_LENGTH)/1024/1024,2), 'MB') as data
 FROM information_schema.tables
 WHERE TABLE_SCHEMA='DBName'
 ORDER BY DATA_LENGTH+INDEX_LENGTH desc;
```

- 查看用户授权情况

```sql
SHOW GRANTS for UserName
# 全局授权
SELECT * FROM mysql.user where User=Username\G
# DB授权
SELECT * FROM mysql.db where User=Username\G
```
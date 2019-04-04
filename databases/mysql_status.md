# MySQL调试命令概要

- 查看当前MySQL进程。如果负载过高，可以通过它查出那些进程在等待，优化SQL

        show processlist;

- 查看所有状态信息

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
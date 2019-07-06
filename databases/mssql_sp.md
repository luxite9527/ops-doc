# SQLServer常用系统函数

## Database相关

```sql
# 查看数据库实例所有信息，主要有数据名称，大小，所有者，ID等
exec sp_helpdb
# 重命名数据库实例
exec sp_renamedb 'old_name','new_name'
# 显示表结构
exec sp_help table_name
# 重命名表
exec sp_rename 'old_name','new_name'
# 查看表空间占用物理空间
exec sp_spaceused table_name
```

## Index相关

```sql
# 查看表索引
exec sp_helpindex table_name;
# 索引重命名
exec sp_rename  'table_name.index_name','table_name.new_index_name';
# 删除指定表索引
drop index table_name.index_name;
# 检查表索引的碎片信息
dbcc showcontig(table_name,index_name);
# 重整（重组）指定数据库，表的索引碎片
dbcc indexdefrag(db_name,table_name,index_name);
# 查看表的自动统计信息，返回索引的最近自动统计日期
exec sp_autostats 'tabl_name';
# 更新表中的全部索引统计信息
update statistics table_name;
```

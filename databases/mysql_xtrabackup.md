# MySQL备份恢复

## 基于xtrabackup备份

```bash
xtrabackup --backup --parallel=4 --compress  --compress-threads=4 --user=USERNAME --password=PASSWORD --database=DBNAME --target-dir=/data/backup
```

## 基于xtrabackup恢复

因为`xtrabackup`执行的是物理备份，所以恢复时只需要将备份文件复制回去即可:

```bash
# 先将压缩文件解压,解压命令需要qpress支持.xtrabackup源中有该命令
yum install -y qpress
xtrabackup --decompress --target-dir=/data/backup
xtrabackup --prepare --target-dir=/data/backup/
# 将解压来的文件还原到MySQL数据目录内。会自动读取/etc/my.cnf文件内的内容，找到MySQL的数据存放目录。还原之前，该目录必须 为空。不然会提示文件不为空的失败信息
xtrabackup  --copy-back  --target-dir=/data/backup
# 更改所有者，不然会启动失败
chown -R mysql:mysql /data/mysql
# 启动服务
systemctl start mysqld
```
## 单表备份还原

单表备份时，必须使用`innobackupex`命令，不能使用`xtrabackup`

```bash
# 先完整备份一次
innobackupex --user=backup --password=123456 --include="cumcm.cumcm_sys_country" /home/backup

# 完整备份完之后，再应用Redo日志，怕备份期间有事务产生
innobackupex --apply-log --export /home/backup/2019-0o-10_13-17-57/

# 在要还原的DB上，需要先创建表结构
mysql>CREATE TABLE cumcm.cumcm_sys_country LIKE cumcm.cumcm_sys_country_20190101;

# 取消原来的表空间
mysql>ALTER TABLE cumcm.cumcm_sys_country DISCARD TABLESPACE;

# 停止MySQL服务器
systemctl stop mysqld

# 复制之前备份的文件至MySQL的DATA目录
cp /home/backup/2019-0o-10_13-17-57/cumcm/cumcm_sys_country.* /var/lib/mysql/cumcm
# 提示是否覆盖原来的.frm文件。选择是

# 启动MySQL
systemctl start mysqld

# 导入表空间
mysql>ALTER TABLE cumcm.cumcm_sys_country IMPORT TABLESPACE;

# 检查, 计数应和备份前的一致
SELECT COUNT(*) FROM cumcm.cumcm_sys_country;
```
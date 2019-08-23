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
xtrabackup  --decompress --target-dir=/data/backup
# 将解压来的文件还原到MySQL数据目录内。会自动读取/etc/my.cnf文件内的内容，找到MySQL的数据存放目录。还原之前，该目录必须 为空。不然会提示文件不为空的失败信息
xtrabackup  --copy-back  --target-dir=/data/backup
# 更改所有者，不然会启动失败
chown -R mysql:mysql /data/mysql
# 启动服务
systemctl start mysqld
```

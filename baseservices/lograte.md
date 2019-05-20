# Linux logrotate配置项解决

`logrotate`用于日志轮转。默认每天执行一次，在`/etc/cron.daily/`目录下有一个脚本文件。

要进行日志轮转的配置文件放在`/etc/logrotate.d`目录下即可

```conf
# 日志路径
/usr/local/nginx/logs/*.log {  
su cheng cheng         #针对Ubuntu下的应用。可能没权限。这样就以该用户权限执行
daily                  #每天执行,monthly,weekly
dateext                #轮转完成后，日志后缀加上日期
missingok              #日志文件不存在也执行
rotate       7         #轮转七天之内的日志。即日志将保留从今天开始加七天日志。
compress               #对日志进行压缩
delaycompress          #再下一次轮转时对轮转后日志压缩
notifempty             #在日志为空时不进行轮转
sharedscripts          #有多个日志需要轮转时，只执行一次脚本
postrotate             #日志轮转之后要执行的脚本
     /usr/local/nginx/sbin/nginx -s reload
endscript
}
```

编写完成后，可以进行测试语法是否有问题：

```bash
# 会马上执行一次，以后每天就会执行一次，如果不运行此命令，将不会生效
logrotate -f /etc/logrotate.d/nginx.conf
```

因为默认的每天执行是凌晨三点左右。这样就会导致每天轮替的日志并不能包含完整的一天结束。

```bash
rm -rf /etc/cron.daily/logrotate  
vim /etc/crontab
59 23 * * *　/usr/sbin/logrotate /etc/logrotate.conf
```
# NGINX常见问题处理

## NGINX Reload不生效

```bash
kill -USR2 `cat /var/run/nginx.pid`
kill -QUIT `cat /var/run/nginx.pid.oldbin`
```
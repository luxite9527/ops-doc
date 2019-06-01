# NGINX常见问题处理

## NGINX Reload不生效

```bash
kill -USR2 `cat /var/run/nginx.pid`
kill -QUIT `cat /var/run/nginx.pid.oldbin`
```

## NGINX Location规则

### 语法

    location [=|~|~*|^~|@]  /url/  { ... }

### 优先级

`=`  严格匹配，如果查询匹配，停止搜索，并立即处理这个请求

`^~` 表示普通匹配完后，不再匹配正则，按最精确普通

`~`  正则匹配，区分大小写

`~*` 正则匹配，不区分大小写

`@`  内部重定向，实现内部转发

        location的匹配顺序是先匹配普通，再匹配正则。也就是普通的location匹配完了，还要继续搜索正则的location，一旦匹配到正则的location，就不再匹配后续的正则了。如果有匹配的正则location,那么就会用正则匹配的结果代替前面普通最精确匹配的结果。如果匹配正则完成全，没有找到匹配条件，那就以上一步普通匹配最精确的一条为结果。
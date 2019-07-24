# NGINX配置文件解释

```conf
# 以哪个用户运行
user  nobody;
# 让Nginx自动根据CPU核心计算出
worker_processes  auto;
# 单个进程允许打开的文件数
worker_rlimit_nofile 65535;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    # 使用EPOLL
    use epoll;
    # 最大连接数由   worker_connections * work_process 值
    worker_connections  100862;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    # map 用于自己定义变量
    # 定义一个名称为ClientRealIp的变量。将http_x_forwarded_for变量的值按以下规则赋值:，
    # 1. 按正则匹配$http_x_forwarded_for里面的值
    # 2. 如果正则没有匹配，使用default的值。
    map  $http_x_forwarded_for  $ClientRealIp {
            # 正则没匹配到的话，则取$remote_addr的变量
            default $remote_addr;
            # 正则表达式，将分组realip捕获到的值设置为 $realip，并最终赋给$ClientRealIp
            "~^(?P<realip>((\d{1,3}\.){3}\d{1,3}))"   $realip;
      }

    # 定义限制单个IP访问速率。每个IP访问每个URL的频率限制为5次
    # limit_req在使用时，如果要非常准确的限制请求频率。一定要在"/"目录下使用。很难验证其真实效果
    limit_req_zone   $binary_remote_add$uri  zone=rate_zone:5m rate=5r/s;
    # 每个IP的并发限制，限制每个IP最多多少连接数
    limit_conn_zone  $binary_remote_addr zone=binfa:10m;

    # 开启404错误路转
    fastcgi_intercept_errors on;
  
    # 定义日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  access  "$time_iso8601 $request $status  $remote_addr  $upstream_response_time $upstream_addr";

    # 访问日志      日志路径     套用的日志格式
    # 缓冲1kb的日志，后续一起写入磁盘
    access_log   /var/log/nginx/access.log   main  buffer=1k;
    # 同时支持写入远端rsyslog
    # access_log syslog:server=128.0.255.10:1514 json;

    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay    on;

    keepalive_timeout  15;
    client_header_timeout 10;
    client_body_timeout 10;
    send_timeout 10;

    client_header_buffer_size 32k;
    # 客户端最大请求大小
    client_max_body_size 200m;
    large_client_header_buffers 4 32k;

    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    # 缓存配置，将静态文件缓存在Nginx服务器上。当有新请求时，不需要再从后端服务器上去取了
    # 开启从后端被代理服务器的响应内容缓冲，开启后，会把内容保存在 proxy_buffer_size和profxy_buffers指定的缓冲里边
    proxy_buffering         on;
    # 后端服务器连接的超时时间_发起握手等候响应超时时间
    proxy_connect_timeout 5;
    # 连接成功后，等候后端服务器响应时间。也可以看成后端最大处理时间
    proxy_read_timeout 60;
    # 后端服务器数据回传时间，就是在规定时间之内后端服务器必须传完所有的数据
    proxy_send_timeout 5;
    # 设置缓冲区大小，从被代理的后端服务器取得响应内容，会先读取放置到这里
    proxy_buffer_size 128k;
    # 设置缓冲区大小和数量，从被代理的后端服务器取得响应内容，会放置到这里。该值不能大于proxy_buffers_size
    proxy_buffers 4 128k;
    proxy_busy_buffers_size 128k;
    proxy_headers_hash_max_size 512k;
    proxy_temp_file_write_size 128k;
    # 如果客户端断开请求，也保持后端的下载
    proxy_ignore_client_abort on;
    # 如果遇到后端服务器的40X系列错误，Nginx拦截，也就是说404不由应用程序报，而由Nginx报404。
    proxy_intercept_errors  on;
    # inactive=1d 表示这个zone中的缓存文件如果在1天内都没有被访问，那么文件会被cache manager进程删除掉
    # keys_zone 从共享内存中分配一块50MB的内存
    # max_size=500m, 目录/home/nginx_cache最大使用空间
    # proxy_cache_path的路径需要写入权限
    proxy_cache_path /home/nginx_cache  levels=1:2  keys_zone=nginx_cache:50m inactive=1d max_size=500m;

    proxy_http_version 1.1;
    proxy_set_header Connection "";

    # 定义被代理的负载均衡服务器组
    upstream vargo {
      # 算法，默认是轮循
      ip_hash;
      # 默认NGINX与该后端服务器保持10空闲个连接
      # 要配置proxy_http_version  1.1;  
      # proxy_set_header Connection  "";
      keepalive 10;
      # 如果失败次数达到max_fails，那么在接下来的时间内认为服务不可用，默认为10s
      server 10.1.10.17:80   max_fails=3 fail_timeout=6s;
      server 192.168.0.7:80  weight=4    max_fails=3  fail_timeout=60s;
      # 指定该主机为备份服务器，当前面两台服务都不可用时启用
      server 192.168.0.8:80 backup;
    }

    server {
        # 定义主机名，当访问时像Location规则一样起效
        server_name  www.unknowname.win;
        listen       80;
        # 开启用户自定义HTTP headers
        underscores_in_headers on;
        #charset koi8-r;
        # 定义日志，并使用日志格式
        access_log  /var/log/nginx/access.log  main;
        # 如果相同变量，这里会覆盖全局的
        proxy_connect_timeout 3s;
        proxy_read_timeout 60s;

        # 普通locations，千万记住根不要用正则的location，不然后续的location将匹配不到
        location  / {
            # 请求方法为HEAD，关闭访问日志
            if ($request_method = "HEAD") {
              access_log off;
            }
            # css/js/ico/png结尾的静态文件不记录日志
            if ($uri ~ "\.(css|js|ico|png)$") {
              access_log off;
            }
            # 缓存有效期，这个数值其实就是告诉终端用户浏览器的失效时间。更改后要重启生效
            expires 1d;
            # 最大同时请求rate_zone定义的+burst值（15+25）。多了就会返回503。如果请求超时就404
            limit_req zone=rate_zone burst=20 nodelay;
            # 限制单个IP的并发连接数
            limit_conn binfa 5;
            # 当传输完成多少K后开始限速
            limit_rate_after 5000k;
            # 限制传输速度
            limit_rate 200k;

            # 修改请求主机名，防止后端真实服务器根据Hostname进行分流。将Host值设置为$http_host的值。可以看成Host=$http_host
            proxy_set_header Host $host;
            # 客户端一层代理下，这是真实IP
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            # 将自定义的Header，变量名为req_usr传给后端服务器
            proxy_pass_header req_user;
            proxy_set_header X-NginX-Proxy true;
            proxy_pass http://vargo;
            # 自动故障转移，当主机组中的服务器代理时出现以下情况，自动转移到另一台上面
            proxy_next_upstream http_500 http_502 http_503 error timeout invalid_header;
        }

       # 正则locations
       # 静态图片使用Nginx缓存，缓存池在上面http段定义的
       location ~*  ^/static/images {
        proxy_cache nginx_cache;
        add_header expires 2h;
        proxy_cache_valid 200 302 1h;
        # 缓存的数据，每一个小时重新从服务器上取一次
        proxy_cache_key $host$uri$is_args$args;
        proxy_ignore_headers "Cache-Control" "Expires" "Set-Cookie";
        proxy_pass http://testweb;
      }

        # 定义404页面自动跳转回首页。真实使用时，使用一个提示找不到页面后再跳转
        #error_page   500 502 503 504  /50x.html;
        # 定义404页面由/xueshandai-mobile/规则下的404.html来处理。它会匹配locatin
      error_page 404 = /xueshandai-mobile/404.html;

      location = /50x.html {
        # 将$remote_addr变量的IP重新设置。一般用于像阿里云SLB后面时，$remote_addr地址会变成阿里的SLB地址
        # set_real_ip_form 相当于指定阿里SLB的IP地址。
        set_real_ip_from  192.168.2.0/24;
        # remote_addr 该变量的来源设置为X-Forwrded-for
        real_ip_header    X-Forwarded-For;
        # 递归的去除所配置中的可信IP,如果出现了未出现这些IP段的IP，那么这个IP将被认为是用户的IP
        real_ip_recursive on;
        # 如果做访问限制，需要增加一条set_real_ip_from的允许规则。不然全是502
        allow 192.168.2.0/24;
        # 再加一条允许的单独IP规则
        allow 128.22.189.11/32;
        deny all;
        root   html;
      }
  }
}
```

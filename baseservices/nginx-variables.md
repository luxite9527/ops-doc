# NGINX常用内置环境变量

更权威肯定是[官网](http://nginx.org/en/docs/varindex.html)

nginx的配置文件中可以使用的内置变量以美元符$开始，也有人叫全局变量。其中，部分预定义的变量的值是可以改变的。

$arg_PARAMETER

    这个变量值为：GET请求中变量名PARAMETER参数的值。

$args

    这个变量等于GET请求中的参数。例如，foo=123&bar=blahblah;这个变量只可以被修改
$binary_remote_addr

    二进制码形式的客户端地址。
$body_bytes_sent

    传送页面的字节数
$content_length

    请求头中的Content-length字段。
$content_type

    请求头中的Content-Type字段。
$http_cookie

    COOKIE的值。
$document_root

    当前请求在root指令中指定的值。
$uri

    请求中的当前URI(不带请求参数，参数位于$args)，不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。不包括协议和主机名，例如/foo/bar.html

$document_uri

    与$uri相同。
$http_user_agent

    用户浏览器代理名称
$request

    请求的URL和HTTP协议
$http_referer

    那个页面连接过来的
$host

    请求中的主机头(Host)字段，如果请求中的主机头不可用或者空，则为处理请求的server名称(处理请求的server的server_name指令的值)。值为小写，不包含端口。
$hostname

    机器名使用gethostname系统调用的值
$http_HEADER

    HTTP请求头中的内容，HEADER为HTTP请求中的内容转为小写，-变为_(破折号变为下划线)，例如：$http_user_agent(Uaer-Agent的值), $http_referer...;
$sent_http_HEADER

    HTTP响应头中的内容，HEADER为HTTP响应中的内容转为小写，-变为_(破折号变为下划线)，例如： $sent_http_cache_control, $sent_http_content_type...;
$is_args

    如果$args设置，值为"?"，否则为""。
$limit_rate

    这个变量可以限制连接速率。
$nginx_version

    当前运行的nginx版本号。
$query_string

    与$args相同。
$remote_addr

    客户端的IP地址。
$remote_port

    客户端的端口。
$remote_user

    已经经过Auth Basic Module验证的用户名。
$request_filename

    当前连接请求的文件路径，由root或alias指令与URI请求生成。
$request_body

    这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义。
$request_body_file

    客户端请求主体信息的临时文件名。
$request_completion

    如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空。
$request_method

    这个变量是客户端请求的动作，通常为GET或POST。包括0.8.20及之前的版本中，这个变量总为main request中的动作，如果当前请求是一个子请求，并不使用这个当前请求的动作。
$request_uri

    这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI。
$scheme

    所用的协议，比如http或者是https，比如rewrite ^(.+)$ $scheme://example.com$1 redirect;
$server_addr

    服务器地址，在完成一次系统调用后可以确定这个值，如果要绕开系统调用，则必须在listen中指定地址并且使用bind参数。
$server_name

    服务器名称。
$server_port

    请求到达服务器的端口号。
$server_protocol

    请求使用的协议，通常是HTTP/1.0或HTTP/1.1。

$http_x_fordwarded_for

    用户真实IP与经过的代理IP列表。最前面是用户真实IP。如。1.1.1.1,2.2.2.2,3.3.3.3。只有1.1.1.1是真实的用户IP地址。
$request_time

    请求处理的时间。从接收到客户端请求到正常返回的时间
$upstream_addr

    后端地址
$upstream_response_time

    后端处理的时间
$cookie_x

    cookie值，x为cookie的名称
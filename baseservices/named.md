# 基于bind的DNS主备服务搭建指南

## 基础概念

### 正解

    将主机名解析为IP地址。如将www.qq.com解析至1.1.1.1。这是一般应用场景

### 反解

    将IP解析为主机名。如上面将1.1.1.1解析为www.qq.com。较少使用

### MX记录

    接收邮件的主机名称

### CNAM主机别名

    如www.doamin.com与app.domain.com使用的是同一个主机

### SOA

    管理这个域名的七个重要参数

### NS

    管理这个域名的主机名称

### A记录

    域名解析到的IPv4地址

### AAAA记录

    域名解析到的IPv6地址

## 安装配置

```bash
# 直接安装
yum install bind  bind-chroot -y

# 修改默认配置文件
1.vi /etc/named.conf

# 也可以监听在指定IP上，默认是127.0.0.1
listen-on port 53 { any;};  

# 允许任何查询请求，也可以指定网段，如192.168.0.0/16
allow-query     { any; };

# 增加以下，将非本域的请求转发给外部DNS主机
forwarders { 8.8.8.8;4.4.4.4; };

# 因为内部DNS,禁用安全检查
dnssec-enable no;
dnssec-validation no;

# 定义Zone
Zone "unknowname.test" {
    type master;
    file "unknowname.test.named";
    also-notify {192.168.0.6; };
    ; 允许的备DNS同步文件
    allow-transfer  { 192.168.0.6; };
};
```

## 配置本地域信息

```bash
2.vi /var/named/unknowname.test.named

;定义存活时间
$TTL    288600
@        IN          SOA         @         root   (
; 更新序号，每次修改该文件都需要增大该值，以通知备DNS
2006102001
; Refresh   slave服务器的更新时间，即Slave多久主动连接进
28800
; Retry  当Slave主机更新失败，多久再重新更新一次
14400  
; Expire  重复多久后宣告失败，不再更新
720000
;TTL 在没有设置TTL时，这就是相当于TTL
86400
)
@                  IN            NS              dns1.unknowname.test.
@                  IN            NS              dns2.unnknowname.test.
; DNS的A记录一定要是运行named的主机
dns1               IN            A               192.168.0.5 
svn                IN            A               192.168.0.254

# 保存后，将named文件所有者设置为named:named
chown named:named /var/named/unknowname.test.named

# 启动服务
systemctl enable named && systemctl start named
```

## 备DNS设置

```bash
# 安装
yum install bind -y

# 配置
vim /etc/named.conf
# 也可以监听在指定IP上，默认是127.0.0.1
listen-on port 53 { any;};  

# 允许任何查询请求，也可以指定网段，如192.168.0.0/16
allow-query     { any; };

# 增加以下，将非本域的请求转发给外部DNS主机
forwarders { 8.8.8.8;4.4.4.4; };

# 因为内部DNS,禁用安全检查
dnssec-enable no;
dnssec-validation no;

# 定义Zone
Zone "unknowname.test" { 
    type slave;
    file "slaves/unknowname.test.named";
    masterfile-format text;
    ; 指定Master
    masters  { 192.168.0.6; };
};

# 备DNS不需要定义域名文件，直接启动服务，就会同步主上的文件至本地
systemctl enable named && systemctl start named
```

## 注意事项

 1. 每个Zone至少要一个NS 才行

 2. 后面的分号是英文的分号注释。有可能复制时会报错。改成了中文的分号

 3. 文件的所有者是named:named。如果解析不生效，检查文件所有者是否正确
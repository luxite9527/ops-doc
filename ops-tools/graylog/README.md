# Graylog日志收集服务

## 介绍

Graylog是一个集中日志收集与检索系统。可以将它看成是一个EFK组合。它的底层搜索提供也是Elasticsearch。相比EFK组合它有以下优势:

  1. 日志展示没有5000条的限制。我们知道默认Kibana在搜索页，针对同一条件，最多只能展示5000条。如果要展示更多，只能更改搜索条件或者时间轴
  2. 更多输入类型，支持`GELF,Rsyslog,AMQP,Kafka`等，官方推荐为GELF格式日志

  3. 用户认证功能，可以集成LDAP轻目录认证
  4. Elasicsearch索引自动管理。Graylog引入索引集（Index Set）概念。默认当某一个索引的文档超过200000时，自动进行切割，创建新的索引。从而可以加快检索速度。另外，自动管理索引集里面的索引个数，默认保留最多保留20个索引，然后轮替删除老的索引。这些参数都是可以调整的。
  5. 引入流(Stream)概念，可以根据收集到的日志的条件过滤引入不同流，使用不同索引。并在基于该技术基础之上，实现日志告警功能。比如匹配输出日志的内容，匹配`ERROR`进行告警。
  6. 支持基于匹配日志告警功能，目前支持邮件与回调方式告警。
  7. `Pipeline`功能，该功能属于高级功能，用于对输入的日志内容再进行加工。
  8. 其他高级功能，请查阅[官方文档](http://docs.graylog.org/en/stable/)

## 安装

## Allinone方式

编辑docker-compose.yml文件

```yaml
version: '2'
services:
  # MongoDB: https://hub.docker.com/_/mongo/
  mongodb:
    image: mongo:3
    restart: always
    container_name: graylog-db
    volumes:
      - ./mongo_data:/data/db
  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.6.1
    restart: always
    container_name: graylog-es
    volumes:
      - /etc/localtime:/etc/localtime
      - ./elas_data:/usr/share/elasticsearch/data
    environment:
      - http.host=0.0.0.0
      - transport.host=0.0.0.0
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
  # Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:3.0
    restart: always
    container_name: graylog-server
    environment:
      # echo -n "admin" | sha256sum
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      # IP设置成宿主机IP
      - GRAYLOG_HTTP_EXTERNAL_URI=http://128.0.255.104:9000/
      - GRAYLOG_TRANSPORT_EMAIL_ENABLED=true
      - GRAYLOG_TRANSPORT_EMAIL_HOSTNAME=smtp.qq.com
      - GRAYLOG_TRANSPORT_EMAIL_PORT=25
      - GRAYLOG_TRANSPORT_EMAIL_USE_AUTH=false
      - GRAYLOG_TRANSPORT_EMAIL_USE_TLS=false
      # 用以邮件告警，填写真实相关信息
      - GRAYLOG_TRANSPORT_EMAIL_AUTH_USERNAME=username
      - GRAYLOG_TRANSPORT_EMAIL_AUTH_PASSWORD=password
      - GRAYLOG_ROOT_TIMEZONE=Asia/Shanghai
      - "JAVA_OPTS=-Xms512m -Xmx512m"
    links:
      - mongodb:mongo
      - elasticsearch
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 514:514/udp
      # Syslog Raw
      - 5555:5555/udp
      # GELF UDP
      - 12201:12201/udp
```

因为docker-compose的原因，新生成的卷权限为root，还需要做点特殊处理

```bash
# 先启动服务
docker-compose up -d
# 启动服务，会在当前目录会创建卷目录,给mongo卷给予权限
chown 777 -R mongo_data
# 再重新启动服务就会正常了
docker-compose restart
```

## 官方优化建议

  1. graylog-server节点，焦点是CPU，因为它是CPU密集型。所以CPU尽量要够强

  2. Elasticsearch节点，尽量给予可用的RAM，因为整个Graylog的响应速度取决于它的I/O
  3. MongoDB，只是用来存储配置信息以及过期的信息,性能一般即可。
  4. graylog Web组件主要用于响应与应用HTTP请求，一般对待即可。
  5. 在运行elasticsearch的节点上，修改进程打打开的文件描述符，至少为64000。
  6. 修改elasticsearch/bin/elasticsearch中的`ES_HEAP_SIZE`，尽量给予更多的可用内存
  7. 在SSD或者SAN存储上运行Elasticsearch时。修改`elasticsearch.yml`
`indices.store.throttle.max_bytes_per_sec: 150mb`

## 管理端初始化

服务起来以后，就需要创建`Input`,`Input`是输入源的类型定义。一般常用的有以下:

登入系统，进入"System/Inputs",创建`Input`类型:
![创建Input](images/graylog-input.jpg)

- GELF 这是官方推荐的一种日志格式，这种输入类型一般我们应用在应用程序里面，应用程序在打印日志的时候，将日志格式设置为GELF格式，并通过UDP发送过来。
- Rsyslog UDP 一般用于配合Linux的Rsyslog，将操作系统的日志收集过来
- Raw/Plaintext UDP 纯UDP数据包，可以用于此类型。
  
要创建的Input根据要收集的日志类型不同而创建。
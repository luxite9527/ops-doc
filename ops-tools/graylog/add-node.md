# Graylog增加节点

如果要完全实现高可用，那么`Graylog`三个组件都需要做到。官方并没有给出具体实施细节，但是官方给出了指导方案

## MongoDB

    需要做成复制集，并且使用Graylog连接MongoDB的URI修改成复制集的连接方式

## Elasticsearch

    Elasticsearch原生就是分布式的，没什么难度。增加discovery.zen.ping.unicast.hosts属性，指定节点组成集群

## Graylog Server

    多个Graylog时，只能有一个Master。

## 可用`YAML`

```yaml
# 这里不做Mongo复制集，所以不需要运行MongoDB
# version: "3.3"
# MongoDB: https://hub.docker.com/_/mongo/
#mongodb:
#  image: mongo:3
#  restart: always
#  container_name: graylog-db
#  net: host
#  volumes:
#    - ./mongo_data:/data/db
#  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docker.html
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.6.1
  restart: always
  container_name: graylog-es
  net: host
  volumes:
    - /etc/localtime:/etc/localtime
    - ./elas_data:/usr/share/elasticsearch/data
    - ./plugins:/usr/share/elasticsearch/plugins
  environment:
    - http.host=0.0.0.0
    - transport.host=0.0.0.0
    - network.host=0.0.0.0
    - "ES_JAVA_OPTS=-Xms3000m -Xmx3000m"
    # 增加ENV，指定前一个ES,这样ES组成集群
    - "discovery.zen.ping.unicast.hosts=128.0.255.10"
  ulimits:
    memlock:
      soft: -1
      hard: -1
  mem_limit: 4g
# Graylog: https://hub.docker.com/r/graylog/graylog/
graylog:
  image: graylog/graylog:3.0
  restart: always
  net: host
  container_name: graylog-server
  environment:
    - TZ=Asia/Shanghai
    - GRAYLOG_ELASTICSEARCH_HOSTS=http://localhost:9200
    # 注意，这里的Mongo使用第一个的。因为要配置共享
    - GRAYLOG_MONGODB_URI=mongodb://128.0.255.10:27017/graylog
    # 不指定为Master
    - GRAYLOG_IS_MASTER=false
    # echo -n "admin" | sha256sum
    - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
    # IP设置成主Graylog的API地址
    - GRAYLOG_HTTP_EXTERNAL_URI=http://128.0.255.10:9000/
    - GRAYLOG_TRANSPORT_EMAIL_ENABLED=true
    - GRAYLOG_TRANSPORT_EMAIL_HOSTNAME=smtp.sina.com
    - GRAYLOG_TRANSPORT_EMAIL_PORT=25
    - GRAYLOG_TRANSPORT_EMAIL_USE_AUTH=true
    - GRAYLOG_TRANSPORT_EMAIL_USE_TLS=false
    # 用以邮件告警，填写真实相关信息
    - GRAYLOG_TRANSPORT_EMAIL_AUTH_USERNAME=user@domain
    - GRAYLOG_TRANSPORT_EMAIL_AUTH_PASSWORD=user@pass
    - GRAYLOG_TRANSPORT_EMAIL_SUBJECT_PREFIX=GraylogAlert
    - GRAYLOG_TRANSPORT_EMAIL_FROM_EMAIL=user@domain
    # IP设置成主Graylog的API地址
    - GRAYLOG_TRANSPORT_EMAIL_WEB_INTERFACE_URL=http://128.0.255.10:9000/
    - GRAYLOG_ROOT_TIMEZONE=Asia/Shanghai
    - "JAVA_OPTS=-Xms512m -Xmx512m"
```

然后执行

```bash
docker-compose up -d
chown -R 777 elas_data
docker-compose restart
```

## 检查

1. 调用ES的`http://es:9200/_cluster/state?pretty`HTTP API，确认新加入的节点已经位于集群中

2. 进入Graylog的WEB，进入`System`-->`nodes`,正常应该会显示二个节点
3. 往新节点输入日志，应该可以在主节点的WEB上展示出来

## 客户端配置

 两台主机都可以接受日志了，并且可以统一展现出来。这样增加吞吐量
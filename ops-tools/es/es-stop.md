# Elasticsearch正确关闭维护

## 通过API，关闭全局负载均衡

```bash
curl -XPUT http://es:9200/_cluster/settings -d '{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"
    }
 }'
```

## 停止`Elasticsearch`服务或者进程

```bash
systemctl stop elasticsearch
```

### 重新打开全局负载均衡

```bash
curl -XPUT http://es:9200/_cluster/settings -d '{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
 }'
```
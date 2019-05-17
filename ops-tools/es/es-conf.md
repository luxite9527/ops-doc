# Elasticsearch 重要的配置项

```yml
# 默认数据副本数
index.number_of_replicas: 1
# 默认索引分片数，分片数X副本数就是这个集群的最佳主机数
# 每个节点一个分片，每个分片可以100%使用主机的资源(IO/CPU/Memry)
# 主分片数一旦确定后就不可更改
index.number_of_shards: 5
# 达到该阈值，就会把旧数据交换出去。该参数可以设置百分比或者绝对值
# EFK场景一定要设置该值,不然会导致运行一段时间后就不能查询
indices.fielddata.cache.size： 60%
# 索引的刷新频率，默认1秒，太小会造成索引频繁刷新，新的数据写入就慢了。（此参数的设置需要在写入性能和实时搜索中取平衡）通常在ELK场景中需要将值调大一些比如60s，在有_template的情况下，需要设置在应用的_template中才生效。
index.refresh_interval: 10s
# 服务监听地址，默认为127.0.0.1
network.host：172.16.1.1
# 一启动就占用分配的内存，防止使用SWAP分区
# ES_HEAP_SIZE 这个JAVA_OPTS官方建设设置为物理内存的一半
bootstrap.memory_lock: true
```
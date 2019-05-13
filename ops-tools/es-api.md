# Elasticsearch常用API

## Index API

- 查看所有索引
  
        curl -XGET http://es:9200/_cat/indices?pretty

- 删除指定索引
  
        curl -XDELETE http://es:9200/IndexName

## 集群API

- 查看集群健康状况

        curl -XGET http://es:9200/_cluster/health?pretty

- 查询集群详细信息
  
        curl -XGET http://es:9200/_cluster/state?pretty

- 查看集群的节点详细信息

        curl -XGET http://es:9200/_nodes/NodeName?pretty

- 查看阻塞的任务
  
        curl -XGET http://es:9200/_cluster/pending_tasks?pretty
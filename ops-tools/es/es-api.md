# Elasticsearch常用API

## Index API

- 查看所有索引
  
        curl -XGET http://es:9200/_cat/indices?pretty

- 删除指定索引
  
        curl -XDELETE http://es:9200/IndexName

- 索引配置信息
  
        curl -XGET http://es:9200/IndexName/_settings?pretty

## 集群API

- 修改默认密码

        curl -XPUT -u elastic  'http://es:9200/_xpack/security/user/elastic/_password'  -d '{
                "password" : "NewPassword"
        }'

- 查看集群健康状况

        curl -XGET http://es:9200/_cluster/health?pretty

- 查询集群详细信息
  
        curl -XGET http://es:9200/_cluster/state?pretty

- 查看集群的节点详细信息

        curl -XGET http://es:9200/_nodes/NodeName?pretty

- 查看阻塞的任务
  
        curl -XGET http://es:9200/_cluster/pending_tasks?pretty

## 模板API

- 查看所有模板
  
        curl -XGET http://es:9200/_template?pretty

- 查看模板详细信息

        curl -XGET http://es:9200/_template/TemplateName?pretty

## 搜索 API

- 检索信息

        curl -XGET http://es:9200/_search?pretty

- 检索指定索引的信息

        curl -XGET http://es:9200/IndexName/_search?pretty

- 检索指定索引指定Type的信息

        curl -XGET http://es:9200/IndexName/TypeName/_search?pretty
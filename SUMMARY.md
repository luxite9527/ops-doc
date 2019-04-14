# 目录

* [前言](README.md)
  
* [基础服务](baseservices/README.md)
  
  * [DNS](baseservices/named.md)
  * NGINX
    * [NGINX常用内置变量](nginx-variables.md)

* [数据库](database/README.md)
  
  * [MySQL Debug](databases/mysql_status.md)

* [生态工具](ops-tools/README.md)
  * [GitLab CI/CD](ops-tools/gitlab-ci.md)
  * GrayLog日志收集
    * [介绍](ops-tools/graylog/README.md)
    * [实践-收集Linux系统日志](ops-tools/graylog/linux.md)
    * [实践-收集Nginx访问日志](ops-tools/graylog/nginx.md)
    * [实践-基于Stream与Rule实现日志告警](ops-tools/graylog/alert.md)
  * Ceph分布式文件系统
    * [介绍](ops-tools/ceph/README.md)
    * [简单调试](ops-tools/ceph/ceph-debug.md)
    * [实践-K8S基于Rook部署Ceph](ops-tools/ceph/k8s-install-ceph.md)
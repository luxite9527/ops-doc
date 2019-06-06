# CoreDNS 配置示例

```conf
.:53 {
        hosts {
          172.18.171.110  a.test2.io
          172.18.203.237  a.test2.io
          ttl 3600
          reload 1m
          fallthrough
        }
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
        forward test.io 172.18.171.109 172.18.171.113
    }
```

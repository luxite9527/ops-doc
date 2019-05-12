# Ceph CRUHSH Map调试修改

## 获取CRUH Map

```bash
ceph osd getcrushmap  -o compile.cursh  #获取编译后的文件，并输出为compile.cursh
crushtool -d compile.cursh  -o  uncompile.crush  #解码出文本格式可以直接修改
```

## 编译注入新的

```bash
crushtool -c uncompile.cursh  -o new_compile
ceph osd setcrushmap -i new_compile
```

ceph osd crush dump
ceph mds dump
ceph  pg dump
ceph osd dump #显示OSD显示
ceph mon dump
ceph osd crush rule ls  #规则集展示

#定义Rule之后，就可以指定Ceph Pool使用指定的Rule.从而指定数据存放在那个节点上/机架上等。

```bash
# buckets
host node2 {
        id -3           # do not change unnecessarily
        # weight 0.015
        alg straw2
        hash 0  # rjenkins1
        item osd.1 weight 0.015
}
host node3 {
        id -4           # do not change unnecessarily
        # weight 0.015
        alg straw2
        hash 0  # rjenkins1
        item osd.2 weight 0.015
}
host node1 {
        id -2           # do not change unnecessarily
        # weight 0.015
        alg straw2
        hash 0  # rjenkins1
        item osd.0 weight 0.015
}

root default {
        id -1           # do not change unnecessarily
        # weight 0.030
        alg straw2
        hash 0  # rjenkins1
        item node2 weight 0.015
        item node3 weight 0.015
        item node1 weight 0.015
}

root test {
        id -5           # do not change unnecessarily
        # weight 0.015
        alg straw2
        hash 0  # rjenkins1
        item node1 weight 0.015
}

# rules
rule replicated_ruleset {
        ruleset 0
        type replicated
        min_size 1
        max_size 10
        step take default
        step chooseleaf firstn 0 type host
        step emit
}
rule test {
        ruleset 1
        type replicated
        min_size 1
        max_size 10
        step take test  #如果使用这条规则，则会在node1上存储数据。但因为至少复制一份，所以状态是WARN。它只是把数据复制给自己
        step chooseleaf firstn 0 type host
        step emit
}
```

## 示例创建一个新的PG池，并指定Rule为上面的Test

```bash
ceph osd  pool create  test-pool 80 80
ceph osd pool set test-pool crush_ruleset 1  #使用编号，不是名称。就是crushmpa中的定义的编号
```
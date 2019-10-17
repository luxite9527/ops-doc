# K8S集群上部署Ceph

现在都讲究云原生，`Rook`是一个云原生存储开源项目。基于它部署`Ceph`将十分简单方便

- 安装`Ceph`集群。下载`Rook`相应的`Release`包，并解压。`Ceph`部      署支持三种模式，这里以`BlockStorage`为例

    ```bash
    cd cluster/examples/kubernetes/ceph
    kubectl create -f common.yaml
    kubectl create -f operator.yaml
    # 前面二个直接执行，下面的文件有很多注释，在生产环境下，根据实际情况进行调整。生产当中，建议拿几台机器专门跑Ceph存储节点
    kubectl create -f cluster.yaml
    ```

- 准备`StorageClass`

    ```bash
    # 根据实际情况，修改自带的StorageClass相关属性
    vim cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml
    ```

- 声明`PVC`
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: data-claim
    spec:
        storageClassName: rook-ceph-block
        accessModes:
        - ReadWriteOnce
        resources:
            requests:
                storage: 2Gi
    ```
    ```bash
    kubectl apply -f pvc.yaml
    ```

- `Deployment`应用`PVC`

  ```yaml
  ......
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /data
      name: data-volume
  ......
  terminationGracePeriodSeconds: 3
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: data-claim
  ```

  ## 额外组件

  ### Dashboard

  默认是启用`Dashboard`的，除非你禁用了。系统会创建`Dashboard`相关的`Pod`与`Service`。但如果要外部能访问，还是要做一点事情的.

  - 暴露`Service`使外部可以访问

    ```bash
    cd cluster/examples/kubernetes/ceph
    kubect apply -f dashboard-external-https.yaml
    kubectl get svc -n rook-ceph 
    ```

  - 获取`Dashboard`的密码

    ```bash
    kubectl get secret rook-ceph-dashboard-password \
     -n rook-ceph -o jsonpath="{['data']['password']}" \
     | base64 --decode && echo
    ```
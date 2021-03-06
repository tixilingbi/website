+++
title = "日志部署"
description = "日志部署"
weight = 26
+++

## 前置要求与约定

日志作为独立的一部分，如果您选择安装，需要准备额外的资源：

- 内存: 12G及以上(3个节点每个节点空闲4G以上)
- 磁盘: ssd或高速存储介质50G及以上(根据实际情况增加磁盘)
- CPU: 2核2线程及以上

## 部署日志组件

<blockquote class="note">
日志非平台运行的必要组件，但如果需要安装调用链则需要安装。安装日志组件，可以统一查看日志和搜索日志。
</blockquote>

### 添加choerodon chart仓库

```
helm repo add c7n https://openchart.choerodon.com.cn/choerodon/c7n/
helm repo update
```

### 安装日志组件

- 创建存储卷pv和pvc

    ```bash
    helm install c7n/create-pv \
        --set type=nfs \
        --set pv.name=elasticsearch-pv\
        --set nfs.path=/u01/elasticsearch \
        --set nfs.server=nfs.example.com \
        --set pvc.name=elasticsearch-pvc \
        --set size=100Gi \
        --set accessModes={ReadWriteMany} \
        --name elasticsearch-pv --namespace=logging
    ```

- 为nfs相关文件授权

    ```bash 
    chmod -R 777 /u01/elasticsearch
    ```

<blockquote class="note">
创建pvc时注意在nfs中创建对应的目录
</blockquote>

- 安装Elasticsearch

    ```bash
    helm install c7n/elasticsearch \
        --set data.storageType=nfs \
        --set replicaCount=3 \
        --set minimumMasterNodes=2 \
        --set persistence.enabled=true \
        --set persistence.existingClaim=elasticsearch-pvc \
        --set clusterName=example \
        --set service.enabled=true \
        --name=elasticsearch \
        --namespace=logging
    ```

- 安装日志收集服务

    ```bash
    helm install c7n/choerodon-logging \
        --set fluentd.elasticsearch.host="elasticsearch-elasticsearch.logging" \
        --namespace=logging \
        --name=choerodon-logging \
    ```

- 安装kibana

    ```bash
    helm install c7n/kibana \
        --set elasticsearch.host="elasticsearch-elasticsearch.logging" \
        --set service.enabled=true \
        --set ingress.enabled=true \
        --set ingress.host=kibana.example.com \
        --namespace=logging \
        --name=kibana
    ```

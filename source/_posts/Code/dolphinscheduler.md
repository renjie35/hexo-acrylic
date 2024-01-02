---
title: DolphinScheduler Kubernetes Deployment
date: '2023-11-25'
categories: Kubernetes
keywords: 'Dolphinscheduler, Kubernetes, Helm, K8s'
description: 使用 Helm 在 Kubernetes 上部署 DolphinScheduler 的详细步骤。
cover: /images/dolphinscheduler.png
tags:
  - Dolphinscheduler
  - Kubernetes
  - K8s
  - Helm
abbrlink: 4c88679b
---
## 先决条件

- [Helm](https://helm.sh/) 3.1.0+
- [Kubernetes](https://kubernetes.io/) 1.12+
- PV 供应(需要基础设施支持)

```3.1.5
tar -zxvf apache-dolphinscheduler-3.1.5-src.tar.gz
cd apache-dolphinscheduler-3.1.5-src/deploy/kubernetes/dolphinscheduler
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add bitnami-full-index https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami

helm dependency update .
helm install dolphinscheduler . --set image.tag=3.1.8


kubectl port-forward --address 0.0.0.0 svc/dolphinscheduler-api 12345:12345
```

修改values.yaml配置

将

```
persistentVolumeClaim:
	storageClassName: "-"
```

改为

```
persistentVolumeClaim:
	storageClassName: "openebs-hostpath"
```



若配置错误，则卸载后重新安装

```
helm uninstall dolphinscheduler
kubectl delete pvc -l app.kubernetes.io/instance=dolphinscheduler
```



## 创建命名空间

```
kubectl create namespace dolphinscheduler
```



## 问题

1.无法从bitnami更新，采用本地托管方式

```
单独下载
https://raw.githubusercontent.com/bitnami/charts/archive-full-index/bitnami/index.yaml
通过docker启动服务
docker run -p 80:8080 -v `pwd`/:/app bitnami/nginx
helm repo add bitnami-full http://localhost
```



2.无法创建租户--存储问题

2.1 Master、Worker 和 Api 服务之间支持共享存储？

```yaml
# 修改values.yaml
common:
  sharedStoragePersistence:
    enabled: true
    mountPath: "
    /opt/soft"
    accessModes:
      - "ReadWriteMany"
    storageClassName: "-"
    storage: "20Gi"
```

`storageClassName` 和 `storage` 需要被修改为实际值

> **注意**: `storageClassName` 必须支持访问模式: `ReadWriteMany`

1. 将 Hadoop 复制到目录 `/opt/soft`
2. 确保 `$HADOOP_HOME` 和 `$HADOOP_CONF_DIR` 正确



2.2 如何支持本地文件存储而非 HDFS 和 S3？

```yaml
# 修改values.yaml
...
resource.storage.upload.base.path: /tmp/dolphinscheduler
...
resource.hdfs.fs.defaultFS: file:///
```



## 参考部署方案

![image-20231201132837643](/Users/ren/Library/Application Support/typora-user-images/image-20231201132837643.png)

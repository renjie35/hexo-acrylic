---
title: Kubernetes下的OpenEBS安装与PV供应配置
date: '2023-11-27'
categories: Kubernetes
keywords: 'OpenEBS, Kubernetes, K8s, PV供应, iSCSI, cStor, NFS'
description: 在Kubernetes上安装OpenEBS，并配置PV供应，实现RWX存储。
cover: /images/k8s.png
tags:
  - Kubernetes
  - K8s
  - OpenEBS
  - PV供应
abbrlink: 4828fc0e
---
## 安装openebs PV供应

```
#安装iSCSI
yum -y install iscsi-initiator-utils
systemctl enable iscsid
systemctl start iscsid

#安装openebs
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml

#或者使用cStor（支持RWX）
kubectl apply -f https://openebs.github.io/charts/cstor-operator.yaml

#替换image否则下载不了
registry.k8s.io/sig-storage ->  registry.aliyuncs.com/google_containers
```



## 实现RWX

```
# 下载iscsi
# 在需要作为存储的机器
yum install iscsi-initiator-utils -y

cat /etc/iscsi/initiatorname.iscsi

sudo systemctl enable --now iscsid

systemctl status iscsid
```

```
# 安装openebs cstor
wget https://openebs.github.io/charts/openebs-operator.yaml
wget https://openebs.github.io/charts/cstor-operator.yaml
kubectl apply -f cstor-operator.yaml
```

```
# 安装nfs
# 所有节点
sudo yum install nfs-utils -y

kubectl apply -f nfs-operator.yaml
```


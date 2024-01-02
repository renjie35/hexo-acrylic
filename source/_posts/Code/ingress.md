---
title: Kubernetes Ingress、MetalLB 的部署指南
date: '2023-11-27'
categories: Kubernetes
keywords: 'Kubernetes, K8s, Ingress, MetalLB'
description: 在 Kubernetes 集群中部署 Ingress、MetalLB 的详细步骤。
cover: /images/k8s.svg
tags:
  - Kubernetes
  - K8s
  - Ingress
  - MetalLB
abbrlink: 6019757c
---
## 1.下载并应用
```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.x.x/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f deploy.yaml
```

## 2.新增一个Ingress配置

```yaml
# demo.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myservice
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: dolphinscheduler-api
            port:
              number: 12345
  ingressClassName: nginx

```

```bash
kubectl apply -f demo.yaml
```

## 3.新增MetalLB

### 3.1启用严格 ARP 模式

```bash
kubectl edit configmap -n kube-system kube-proxy
```

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

### 3.2 应用配置文件

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

### 3.3 定义网络ip池

```yaml
# ip_pool.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: web-ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 58.208.85.3-58.208.85.3
```

```bash
kubectl apply -f ip_pool.yaml
```



## 问题

- 只有一台可以访问，其他无法访问

修改spec.externaltrafficpolicy字段，改为Cluster

```yaml
...
spec:
  externalTrafficPolicy: Cluster
...
```

在k8s的Service对象（申明一条访问通道）中，有一个“externalTrafficPolicy”字段可以设置。有2个值可以设置：Cluster或者Local。

1）Cluster表示：流量可以转发到其他节点上的Pod。

2）Local表示：流量只发给本机的Pod。



参考链接：

[externaltrafficpolicy的有关问题说明 ](https://www.cnblogs.com/zisefeizhu/p/13262239.html)

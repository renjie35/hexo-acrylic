---
title: Helm 安装指南
date: '2023-11-27'
categories: Kubernetes
keywords: Helm
description: 使用命令行和下载方式在 Linux 系统上安装 Helm。
cover: /images/helm.svg
tags:
  - Helm
abbrlink: 23f6cb56
---
## 安装

1.根据网址找到对于的地址xxx
https://github.com/helm/helm/releases

```
wget xxx
tar -zxvf xxx.gz
mv linux-amd64/helm /usr/local/bin/helm
```

2.命令行安装
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## 文档地址

https://helm.sh/docs/intro/install/
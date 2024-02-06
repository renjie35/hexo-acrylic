---
title: GitLab在x86和ARM环境下使用docker-compose部署
date: '2023-12-26'
categories: DevOps
keywords: GitLab, Docker, ARM, x86
description: 介绍了如何在x86和ARM环境下配置和启动GitLab服务。
cover: /images/docker.webp
tags:
  - GitLab
  - Docker
---
## x86环境（大多数服务器）
### 配置yaml
``` docker-compose.yml
version: '3.3'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: '192.168.xx.xx'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.xx.xx:4080'
        gitlab_rails['gitlab_shell_ssh_port']=10022
    ports:
      - '4080:4080'
      - '10443:443'
      - '10022:22'
    volumes:
      - './srv/config:/etc/gitlab'
      - './srv/logs:/var/log/gitlab'
      - './srv/data:/var/opt/gitlab'
    shm_size: '256m'
```
### 拉取yaml，启动服务
## arm环境（mac m1,m2）
### 配置yaml
``` docker-compose.yml
version: '3.6'
services:
  web:
    image: 'yrzr/gitlab-ce-arm64v8:latest'
    restart: always
    hostname: '192.168.xx.xx'
    ports:
      - '4080:80'
      - '10443:443'
      - '10022:22'
    volumes:
      - './srv/config:/etc/gitlab'
      - './srv/logs:/var/log/gitlab'
      - './srv/data:/var/opt/gitlab'

```

###修改gitlab.rd
```
vi /etc/gitlab/gitlab.rd
```

``` gitlab.rd
#Gitlab最终的访问地址
external_url 'http://192.168.xx.xx:30080/'
#最终的SSH地址
gitlab_rails['gitlab_ssh_host'] = '192.168.xx.xx' 
#最终访问SSH的端口号
gitlab_rails['gitlab_shell_ssh_port'] = 10022 
#nginx监听地址
nginx['listen_addresses'] = ['*']
#容器内部nginx的监听端口
nginx['listen_port'] = 80
#执行以下命令生效配置
gitlab-ctl reconfigure
```

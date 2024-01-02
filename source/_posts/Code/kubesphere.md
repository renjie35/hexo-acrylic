---
title: k8s部署KubeSphere并开启DevOps
date: '2023-12-20'
categories: KubeSphere
keywords: KubeSphere, DevOps, Kubernetes, K8s
description: 在 Kubernetes 集群中部署 KubeSphere 的详细步骤，并开启DevOps配置。
cover: /images/kubesphere.svg
tags:  
  - KubeSphere 
  - K8s
  - Kubernetes 
  - DevOps
---
## 设置默认的sc

```bash
kubectl patch sc openebs-rwx -p '{"metadata": {"annotations": {"storageclass.beta.kubernetes.io/is-default-class": "true"}}}'
```



## k8s部署kubesphere

下载文件

```bash
wget https://github.com/kubesphere/ks-installer/releases/download/v3.3.2/kubesphere-installer.yaml
wget https://github.com/kubesphere/ks-installer/releases/download/v3.3.2/cluster-configuration.yaml
```

部分存储卷 VolumeSize 小于20G,需要手动调整

```yaml
mysqlVolumeSize: 20Gi # MySQL PVC size.
minioVolumeSize: 20Gi # Minio PVC size.
etcdVolumeSize: 20Gi  # etcd PVC size.
openldapVolumeSize: 2Gi   # openldap PVC size.
redisVolumSize: 2Gi # Redis PVC size.
  elasticsearchMasterVolumeSize: 4Gi   # Volume size of Elasticsearch master nodes.
  elasticsearchDataVolumeSize: 20Gi    # Volume size of Elasticsearch data nodes.
jenkinsVolumeSize: 8Gi     # Jenkins volume size.
prometheusVolumeSize: 20Gi       # Prometheus PVC size.
```

部署yaml

```bash
kubectl apply -f kubesphere-installer.yaml
kubectl apply -f cluster-configuration.yaml

# 查看日志
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
kubectl describe pod kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}')

```

修改宿主机端口

```bash
kubectl edit service ks-console -n kubesphere-system

# 修改type为LoadBalancer或者NodePort
```

开启应用商店和devops

```
vi cluster-configuration.yaml

...
# 开启应用商店
openpitrix:
  store:
    enabled: true # 将“false”更改为“true”
...

...
# 开启devops
devops:
  enabled: true # 将“false”更改为“true”
...

kubectl apply -f cluster-configuration.yaml

#验证
kubectl get pod -n kubesphere-devops-system
```



## #配置devops

新增凭证

- git账号密码
- dockerhub账号密码（**aliyun镜像仓库**）
- kubeconfig（新建项目-配置-服务账号 -- > 创建后再修改角色）

新增流水线，使用以下Jenkinsfile

```yaml
# Jenkinsfile
pipeline {
  agent {
    kubernetes {
      inheritFrom 'nodejs base'
      containerTemplate {
        name 'nodejs'
        image 'node:14.19.0'
      }
    }
  }
  
  stages {
    stage('拉取') {
      agent none
      steps {
        # git拉取代码
        git(credentialsId: 'gitlab-root', url: env.GIT_URL, branch: params.GIT_BRANCH, changelog: true, poll: false)
        
        # 根据选择框，判断TAG_NAME是否填写，若没填写，则自动生成
        script{
          env.GIT_VERSION = sh (script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          if(params.TAG_NAME) {
            env.DOCKER_IMAGE = sh (script: 'echo $TAG_NAME', returnStdout: true).trim()
          } else {
            env.DOCKER_IMAGE = sh (script: 'echo $GIT_VERSION-$BUILD_NUMBER', returnStdout: true).trim()
          }
        }
      }
    }

    stage('编译打包') {
      steps {
        container('nodejs') {
        	# 前台代码的编译打包
          sh 'npm install --registry=https://registry.npm.taobao.org'
          sh 'npm run build'
        }

      }
    }

    stage('构建dockerfile') {
      agent none
      steps {
        container('base') {
        	# 根据项目里的dockerfile生成对应的镜像
          sh 'ls '
          sh 'docker build -t $ALIYUNHUB_STORE:$DOCKER_IMAGE -f $DOCKERFILE_PATH  .'
        }
      }
    }

    stage('推送') {
      agent none
      steps {
        container('base') {
          withCredentials([usernamePassword(credentialsId : 'aliyun-docker' ,usernameVariable : 'DOCKER_USER' ,passwordVariable : 'DOCKER_PASSWORD' ,)]) {
            # 将镜像推送到对应的站点
            sh 'echo $DOCKER_USER | docker login $REGISTRY -u $DOCKER_USER -p $DOCKER_PASSWORD'
            sh 'docker tag $ALIYUNHUB_STORE:$DOCKER_IMAGE $REGISTRY/$ALIYUNHUB_NAMESPACE/$ALIYUNHUB_STORE:$DOCKER_IMAGE'
            sh 'docker push $REGISTRY/$ALIYUNHUB_NAMESPACE/$ALIYUNHUB_STORE:$DOCKER_IMAGE'
          }
        }
      }
    }

    stage('部署') {
      agent none
      steps {
        container('base') {
        	# 根据kubeconfig配置文件，使用对应的项目，部署deploy.yaml
          withCredentials([kubeconfigFile(credentialsId : 'web-kubeconfig' ,variable : 'KUBECONFIG' ,)]) {
            sh 'envsubst < $DEPLOY_PATH | kubectl apply -f -'
          }
        }
      }
    }

  }
  environment {
    GIT_URL = 'http://172.16.100.48:30080/data-platform/web.git'
    
    REGISTRY = 'registry.cn-hangzhou.aliyuncs.com'
    ALIYUNHUB_NAMESPACE = 'zichan'
    
    ALIYUNHUB_STORE = 'data-platform-web'
    DOCKERFILE_PATH = 'docker/Dockerfile'
    DEPLOY_PATH = 'docker/deploy.yaml'
  }
  parameters {
    # 参数设置，可在运行时修改具体的参数
    choice(name: 'GIT_BRANCH', choices: ['master'], description: '请选择要发布的分支')
    string(name: 'TAG_NAME', defaultValue: '', description: '请填写要发布的镜像名（不填自动生成）')
  }
}
```



##附件	

dockerfile

```dockerfile
# docker/dockerfile
FROM nginx

COPY ./dist /data

RUN rm /etc/nginx/conf.d/default.conf

ADD ./docker/default.conf /etc/nginx/conf.d/default.conf

RUN /bin/bash -c 'echo init ok'

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Nignx - default.conf

```nginx
server {
    listen 80;
    server_name localhost;

    gzip on;
    gzip_static on;     # 需要http_gzip_static_module 模块
    gzip_min_length 1k;
    gzip_comp_level 4;
    gzip_proxied any;
    gzip_types text/plain text/xml text/css;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";

    # 前端打包好的dist目录文件
    root /data/;

    # # 若新增后端路由前缀注意在此处添加（|新增）
    # location ~* ^/(dev-api) {
    #    proxy_connect_timeout 15s;
    #    proxy_send_timeout 15s;
    #    proxy_read_timeout 15s;
    #    proxy_set_header X-Real-IP $remote_addr;
    #    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #    proxy_set_header X-Forwarded-Proto http;
    #    rewrite ^/dev-api/(.*)$ /$1 break;
    #    proxy_pass http://url-api:8080;
    # }
}
```

deploy.yaml

```yaml
# docker/deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-vue
  name: web-vue
  namespace: web
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  strategy:
    type: RollingUpdate #  Recreate：在创建新Pods之前，所有现有的Pods会被杀死 RollingUpdate：滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数
    rollingUpdate:
      maxSurge: 1 #maxSurge：1 表示滚动升级时会先启动1个pod
      maxUnavailable: 1 #maxUnavailable：1 表示滚动升级时允许的最大Unavailable的pod个数，也可以填写比例，maxUnavailable=50%
  selector:
    matchLabels:
      app: web-vue
  template:
    metadata:
      labels:
        app: web-vue
    spec:
      containers:
        - env:
            - name: CACHE_IGNORE
              value: js|html
            - name: CACHE_PUBLIC_EXPIRATION
              value: 3d
          image: $REGISTRY/$ALIYUNHUB_NAMESPACE/$ALIYUNHUB_STORE:$DOCKER_IMAGE
          readinessProbe:
            httpGet:
              path: /
              port: 80
            timeoutSeconds: 10
            failureThreshold: 30
            periodSeconds: 5
          imagePullPolicy: Always
          name: web-vue
          ports:
            - containerPort: 80
              protocol: TCP
          resources:
            limits:
              cpu: 300m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 100Mi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      # 私有仓库秘钥，需要在项目中提前建好
      imagePullSecrets:
        - name: ali-docker-registry

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web-vue-service
  name: web-vue-service
  namespace: web
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: web-vue
  sessionAffinity: None
  type: NodePort
```


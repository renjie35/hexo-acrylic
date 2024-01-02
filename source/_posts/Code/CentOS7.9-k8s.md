---
title: CentOS7.9安装k8s | Kubernetes集群搭建
date: '2023-12-29'
categories: k8s
keywords: Kubernetes-k8s-集群
description: 使用kubeadm在CentOS上搭建Kubernetes集群的详细步骤。
cover: /images/css.webp
tags:
  - k8s
abbrlink: 3d4bd17d
---
## 换源（非必要）
```
yum install -y wget
cd /etc/yum.repos.d
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
```

## 基础安装
设置主机名： 

```
hostnamectl set-hostname <hostname>
```

修改host

```
cat >> /etc/hosts << EOF
172.16.100.236  k8s-master
172.16.100.12   k8s-node1
EOF
```

将桥接的IPv4流量传递到iptables的链

```
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

生效
`sysctl --system`

时间同步： 

```
yum install ntpdate -y
ntpdate time.windows.com
```


关闭swap、防火墙和selinux
```
# 关闭swap：(如果使用swap，则其实node的pod使用内存总和可能超过了node的内存)
swapoff -a  # 临时
sudo sed -i '/swap/s/^/#/' /etc/fstab  # 永久

# 关闭防火墙：
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux：(selinux安全机制较复杂，可能会与k8s本身的流量机制冲突，centos与ubunutu可能不同，而且ubuntu部分发行版本默认就没装selinux)
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时
```



## 安装


## 使用阿里云yum源安装kubelet kubeadm kubectl
新增源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
安装指定版本(如非必要别安装最新版本，踩坑得自己埋)
`yum install -y  kubeadm-1.20.2 kubelet-1.20.2 kubectl-1.20.2`
开启kubelet服务
`systemctl enable kubelet.service`


## 安装containerd
```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install containerd.io
```
生成默认配置
`containerd config default > /etc/containerd/config.toml`
配置 systemd cgroup 驱动程序
`sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml`


启动
```
systemctl daemon-reload
systemctl enable containerd --now
systemctl status containerd
systemctl restart containerd
```

## 初始化平面节点(master)
导出 kubeadm 配置文件 

```bash
kubeadm config print init-defaults --component-configs \
KubeProxyConfiguration,KubeletConfiguration > kubeadm-config.yaml
```

替换kubeadm.yml为国内的源

```yaml
...
imageRepository: registry.aliyuncs.com/google_containers
...
```

查看镜像列表

```bash
kubeadm config images list --config kubeadm.yml
```

拉取镜像

```bash
for i in `kubeadm config images list --config kubeadm.yml`;do ctr images pull $i;done
```

初始化

```bash
kubeadm init --upload-certs --apiserver-advertise-address=172.16.100.48 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.23.17 --service-cidr=10.96.0.0/12 --pod-network-cidr=192.168.0.0/16

# --upload-certs 将证书保存到 kube-system 名称空间下名为 extension-apiserver-authentication 的 configmap 中，这样其他控制平面加入的话只要加上 --control-plane --certificate-key 并带上相应的key就可以拿到证书并下载到本地。
# --apiserver-advertise-address 为k8s-master的ip
```

重置

```bash
kubeadm reset
```

初始化集群配置

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



## 安装CNI网络插件，calico flannel cilium 三选1

```主节点
wget https://docs.projectcalico.org/v3.14/manifests/calico.yaml --no-check-certificate

把calico.yaml里pod所在网段改成kubeadm init时选项--pod-network-cidr所指定的网段，
直接用vim编辑打开此文件查找192，按如下标记进行修改：
# no effect. This should fall within `--cluster-cidr`.
# - name: CALICO_IPV4POOL_CIDR
#   value: "192.168.0.0/16"
# Disable file logging so `kubectl logs` works.
- name: CALICO_DISABLE_FILE_LOGGING
value: "true"
把两个#及#后面的空格去掉，并把192.168.0.0/16改成10.96.0.0/12
# no effect. This should fall within `--cluster-cidr`.
- name: CALICO_IPV4POOL_CIDR
value: "10.244.0.0/16"
# Disable file logging so `kubectl logs` works.
- name: CALICO_DISABLE_FILE_LOGGING
value: "true"
```


使用kubectl create 创建相关calico配置

https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart



同步到其他节点

```子节点
scp root@主节点服务器地址:/env/calico/calico.yaml /env/calico/
```



## demo测试创建pod

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc
```





## 错误原因：

- 配置 kubelet 使用 containerd 作为其容器运行时

  vi /var/lib/kubelet/kubeadm-flags.env

  在末尾添加--container-runtime-endpoint=unix:///run/containerd/containerd.sock

  

- [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables] or [ERROR FileContent--proc-sys-net-ipv4-ip_forward]

```
modprobe br_netfilter
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/ipv4/ip_forward
```

- 修改containerd的沙盒配置镜像

```
vi /etc/containerd/config.toml
systemctl daemon-reload && systemctl restart containerd
```

- The connection to the server localhost:8080 was refused - did you specify the right host or port?

```主节点
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```
```子节点
scp root@主节点服务器地址:/etc/kubernetes/admin.conf /etc/kubernetes/
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

- reboot后无法启动k8s，kubelet无法启动

```
vi /var/lib/kubelet/kubeadm-flags.env
删除--network-plugin
```

- K8s Master当作Node使用的方法

```
# Master当作Node使用
kubectl taint node --all node-role.kubernetes.io/master-
# 将 Master 恢复成 Master Only 状态
kubectl taint node --all node-role.kubernetes.io/master="":NoSchedule
```

- docker 拉取问题 missing signature key

```
# 卸载旧的docker
yum erase docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
  
# 新增新的
yum install docker-ce -y
```

  

## 其他插件

安装开启docker

```bash
yum install docker -y
systemctl enable docker && systemctl start docker
```



## 文档地址

https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/

## kubectl命令

```
# 日志journalctl -xefu kubelet

kubectl describe pod xxx 
kubectl logs {podsname} -n xxx

kubectl delete deployment {deploymentname}

kubectl delete pod  {podsname}

kubectl apply -f xxxx.yaml

kubectl delete -f xxxx.yaml

kubectl get namespace
kubectl create namespace xxx
```

列出所有 deployments (别名 `deploy`):

```
kubectl get deploy
kubectl get deploy -n test # with test namespace
```

扩缩容 api 至 3 个副本:

```
kubectl scale --replicas=3 deploy dolphinscheduler-api
kubectl scale --replicas=3 deploy dolphinscheduler-api -n test # with test namespace
```

列出所有 statefulsets (别名 `sts`):

```
kubectl get sts
kubectl get sts -n test # with test namespace
```

扩缩容 master 至 2 个副本:

```
kubectl scale --replicas=2 sts dolphinscheduler-master
kubectl scale --replicas=2 sts dolphinscheduler-master -n test # with test namespace
```

扩缩容 worker 至 6 个副本:

```
kubectl scale --replicas=6 sts dolphinscheduler-worker
kubectl scale --replicas=6 sts dolphinscheduler-worker -n test # with test namespace
```

设置node节点不可调度、驱逐node节点

```
# 不可调度
kubectl cordon <node-name>
# 取消不可调度
kubectl uncordon <node-name>

# 驱逐！！！会将所有节点去除
kubectl drain --ignore-daemonsets <node-name>
# 删除节点
kubectl delete node <node-name>
```

快速回滚

```
#回滚到Deployment上一个版本
kubectl rollout undo <deployment-name> 
#回滚到Deployment到某一个版本,需要先查询版本列表:
kubectl rollout history <deployment-name> 
kubectl rollout undo <deployment-name>  --to-revision=2
```





添加标签并在yaml的**nodeSelector**中使用

```
kubectl label nodes <node-name> disktype=ssd
```

```yaml
...
  nodeSelector:
    disktype: ssd
...
```


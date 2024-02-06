---
title: 初始化新磁盘并挂载
date: '2024-02-01'
categories: Linux
keywords: 磁盘初始化, 挂载, Linux
description: 介绍了在Linux系统下初始化新磁盘并挂载的步骤。
cover: /images/linux.jpg
tags:
  - Linux
  - fdisk
---

## 初始化新磁盘
### 查看分区情况
```
fdisk -l
```

### 新增磁盘
```
fdisk /dev/vdb1
```
n:添加新分区
p:分区类型选择为主分区
w:标识将分区信息写入磁盘

### 格式化磁盘为ext4
```
mkfs.ext4 /dev/vdb1
```

### 挂在磁盘
```
mkdir /data
mount /dev/vdb1 /data
```

### 查看磁盘使用
```
#所有磁盘
df -TH

#当前目录
du -h --max-depth=1
```

### 永久挂载
编辑/etc/fstab文件
添加
``` /etc/fstab
/dev/vdb1 /workspace ext4  defaults  0 0
```

### 来源
https://blog.csdn.net/shenyunsese/article/details/125885311
---
layout: post
title: "ubuntu服务器增加磁盘"
date: 2013-04-03 11:57
comments: true
sharing: true
footer: true
categories: [Share]
---


### 显示硬盘分区

```
sudo fdisk -lu

# 系统显示最下面显示：disk /dev/sdb doesn't contain a valid partition table
```

### 对sdb进行分区

```
sudo fdisk /dev/sdb
```

1. 按m显示帮助文档
2. 根据提示按n 添加一个新分区
3. 按e 添加扩展分区，接下去默认即可
4. 回到帮助文档，按p 显示分区结果
5. 按w 保存分区，并退出完成
6. 可以再fdisk -lu看看，结果已经不同了，硬盘认出来了

<!-- more -->

### 格式化磁盘

+ -t 是文件系统类型

```
sudo mkfs -t ext4 /dev/sdb
```

### 挂载磁盘

```
sudo mount -t ext4  /data

# 查看结果，应该有记录了
sudo df -l
```

### 配置启动挂载

```
sudo blkid /dev/sdb
# 显示 /dev/sdb: UUID="cc6f31aa-9af3-4283-8e52-8d31f5ebbd36" TYPE="ext4"

vim /etc/fstab
# 添加一行
UUID=cc6f31aa-9af3-4283-8e52-8d31f5ebbd36 /data ext4 defaults 0 0
```

+ fstab里的pass 根目录应当获得最高的优先权 1, 其它所有需要被检查的设备设置为 2. 0 表示设备不会被 fsck 所检查。

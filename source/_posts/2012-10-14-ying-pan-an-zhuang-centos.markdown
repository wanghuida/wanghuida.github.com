---
layout: post
title: "硬盘安装CentOS"
date: 2012-10-14 21:48
comments: true
sharing: true
footer: true
categories: [Share]
---


####下载diskgenuis进行分区和设置卷标
+ 【C盘】，NTFS；安装的是win7
+ 【D盘】，卷标：REV，FAT32；

####下载CentOS-6.2-i386-LiveCD.iso
+ 把isolinux下的vmlinuz0和initrd0.img提取到C盘根目录下
+ 把centos 6.2 livecd解压到D盘根目录下;

####下载EasyBCD进行boot设置
+ 使用easybcd安装NeoGrub，配置menu.lst，内容如下：

```
title CentOS LiveCD
#如果前面还有分区设置为root (hd0,1)，设置错误了也没关系，重启后会报错，按e调整，再按b去boot
root (hd0,0) 
kernel /vmlinuz0 root=live:LABEL=REV rootfstype=auto ro liveimg quiet rhgb 
initrd /initrd0.img
boot
```

####重启后，选择新选项即可进入CentOS LiveCD环境。


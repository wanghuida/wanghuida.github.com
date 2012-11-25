---
layout: post
title: "mogilefs设备损坏处理流程"
date: 2012-11-25 19:34
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

+ 这里单个dev对应一个磁盘,假设是sdb
+ 文件系统使用xfs
+ mogstored使用nginx


1.把磁盘标记为dead

```bash
mogadm device mark <hostname> <devid> <status>
```

<!-- more -->

2.格式化新磁盘或损坏的磁盘, 并挂到新的目录

```bash
mkfs.xfs -f -i size=512 -l size=128m,lazy-count=1 -d agcount=16 /dev/sdb

mkdir -p /var/mogdata/{new-dev}

mount -t xfs -o noatime,noikeep,logbufs=8 /var/mogdata/{new-dev} /dev/sdb

#增加到fstab
vim /etc/fstab
/dev/sdb     /var/mogdata/{new-dev}    xfs    noatime,noikeep,logbufs=8    0 0
```

3.配置nginx

```
chown -R mogile.daemon /var/mogdata/{new-dev}

#建议注释掉损坏的location
vim /etc/nginx/conf.d/{xxx.conf}

location /{new-dev}/ {
    expires             max;
    root                /var/mogdata;
    client_max_body_size        20m;
    client_body_temp_path       /var/mogdata/{new-dev}/temp;
    dav_methods                 PUT DELETE MKCOL;
    create_full_put_path        on;
    dav_access                  user:rw group:r all:r;
}

/etc/init.d/nginx reload
```

4.增加一个新设备

```
mogadm device add <hostname> <devid> [opts]        Add a device to a host.

      <devid>              Numeric devid.  Never reuse these.
      <hostname>           Hostname to add a device
      --status=s           One of 'alive' or 'down'.  Defaults to 'alive'.
```

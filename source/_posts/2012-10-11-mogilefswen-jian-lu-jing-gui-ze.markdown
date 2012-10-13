---
layout: post
title: "mogilefs文件路径规则"
date: 2012-10-11 16:16
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

###同事问我mogilefs文件路径的规律是什么
+ 以前有看到过，但是没有记录，一下子就被问悶了，突然发现就记录一下
+ 当fid长度不足10时，前置补0到长度为10,假设补好后的结果为0258693147
+ 通过正则把path切割成0/258/693/147这样
+ 最后的路径结果是：/设备/0/258/693/147/0258693147.fid

```perl
sub uri_path {
    my $self = shift;
    my $devid = $self->{devid};
    my $fidid = $self->{fidid};

    my $nfid;
    my $len = length $fidid;
    if ($len < 10) {
        $nfid = '0' x (10 - $len) . $fidid;
    } else {
        $nfid = $fidid;
    }   
    my ( $b, $mmm, $ttt, $hto ) = ( $nfid =~ m{(\d)(\d{3})(\d{3})(\d{3})} );

    return "/dev$devid/$b/$mmm/$ttt/$nfid.fid";
}
```

###总结一下
+ 这个算法效率很高，并且可以把文件平均分配到目录下

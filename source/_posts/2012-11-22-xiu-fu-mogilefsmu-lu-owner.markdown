---
layout: post
title: "修复mogilefs目录owner"
date: 2012-11-22 17:16
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---


###什么情况会导致mogilefs的数据目录owner不正确

+ 直接使用root启动mogstored会导致增加的文件和文件夹为root
+ 重新用mogile启动mogstored后，mogile用户无法删除root的文件，最终delete队列无法处理


###如何处理

1. 直接chown -R，这是最直接的方法，如果1T的磁盘被这样操作，会10小时util为100
2. 自己写一个脚本慢处理，脚本思路如下

<!-- more -->

```perl
package MtChown;

use strict;
use warnings;
use Data::Dumper;
use Carp;
use Time::HiRes;

sub run {
    my ($dir,$user) = @_;

    if(!defined($dir) || !defined($user)){
        print "please set option [-D|--dir -u|--user]\n"; 
        return 0; 
    }
    $dir =~ s/(.+)\/$/$1/i;

    croak("[$dir] isn't a directory or file") unless (-f $dir or -d $dir);
    my ($login,$pass,$uid,$gid) = getpwnam($user) or die "$user not in passwd file";
    carp("$uid,$gid");
    process($dir,$uid,$gid);
    return 1;
}

sub process {
    my ($dir,$u,$g) = @_;
    if (-d $dir) {
        opendir(FILE,$dir) or croak("opendir error [$dir]");
        my @files = readdir(FILE);
        closedir(FILE);
        foreach (@files) {
            next if ($_ eq '.' || $_ eq '..');
            my $tmp_dir = "$dir/$_";
            if (-d $tmp_dir) {
                process($tmp_dir,$u,$g)
            }else {
                chowner($u,$g,$tmp_dir);
            }
        }
    }
    chowner($u,$g,$dir);
}

sub chowner {
    my ($u,$g,$dir) = @_;
    my ($dev,$ino,$mode,$nlink,$uid,$gid,$rdev,$size,$atime,$mtime,$ctime,$blksize,$blocks) = stat($dir);
    if($u != $uid || $g != $gid){
        my $ret = chown $u,$g,($dir);
        $ret <= 0 ? carp("chown [$u,$g,$dir] error") : carp("chown [$u,$g,$dir] success");
    }
    my $microsec = 50_000;
    Time::HiRes::usleep($microsec);
}

1;
```

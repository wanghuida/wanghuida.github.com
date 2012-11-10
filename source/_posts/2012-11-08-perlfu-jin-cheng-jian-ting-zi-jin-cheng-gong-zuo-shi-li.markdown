---
layout: post
title: "perl父进程监听，子进程工作实例"
date: 2012-11-08 22:15
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---


1. 构造后台进程
2. 主进程监到连接，创建子进程接收消息并处理消息
3. 主进程继续监听，同时非阻塞的回收子进程

<!-- more -->

```perl
#!/usr/bin/perl

use warnings;
use strict;
use IO::Socket;
use Socket qw(SO_KEEPALIVE);
use IO::Poll;
#POSIX标准是必须要的，否则waitpid会阻塞住
use POSIX;
use Carp;
use Errno  qw(EINPROGRESS EWOULDBLOCK EISCONN ENOTSOCK
              EPIPE EAGAIN EBADF ECONNRESET ENOPROTOOPT);

$| = 1;
daemonize();

my $server = IO::Socket::INET->new(LocalAddr => "0.0.0.0:5678",
                                   Type      => SOCK_STREAM,
                                   Proto     => 'tcp',
                                   Blocking  => 0,
                                   Reuse     => 1,
                                   Listen    => 1024 )
    or die "Error creating socket: $@\n";
$server->sockopt(SO_KEEPALIVE, 1); 

while(1) {

    while(my $pid = waitpid -1, WNOHANG) {
        #print "child[$pid] closed\n" if($pid > 0);
        last unless $pid > 0;  
    }

    if(my $csock = $server->accept) {                                                                                                                           
        my $pid = fork;

        if($pid){
            close($csock);
            undef $csock;
        }else{
            $0 .= " [perl child]"; 
            while(1) {
                my $buf;
                my $len = sysread($csock, $buf, 1048576, 0);
                work($buf); 
                last if (!$len && $! != EWOULDBLOCK);
            }
            exit 0;
        }
    }
}

#子进程的工作
sub work{
    print @_."\n";
}


sub daemonize {
    my($pid, $sess_id, $i);
        
    if ($pid = fork) { exit 0; }
        
    croak "Cannot detach from controlling terminal"
        unless $sess_id = POSIX::setsid();
        
    $SIG{'HUP'} = 'IGNORE';
    if ($pid = fork) { exit 0; }
        
    chdir "/";
        
    umask 0;
}

```

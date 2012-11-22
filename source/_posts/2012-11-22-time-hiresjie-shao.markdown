---
layout: post
title: "Time::HiRes介绍"
date: 2012-11-22 17:18
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---




###Time::HiRes库介绍

+ HiRes是high resolution的简写，译为高分辨率
+ HiRes其实是提供一些更高精度的alarm, sleep, gettimeofday, interval timers

<!-- more -->

###usleep，nanosleep实例

+ usleep参数是微妙
+ nanosleep参数是纳秒

```perl
Time::HiRes::usleep(100_000); 
print "100 milliseconds\n";
Time::HiRes::nanosleep(1000_000_000);
print "1 seconds\n"; 
```

###ualarm实例

+ ualarm(0)会撤销警告信号

```perl
eval {      
    local $SIG{ALRM} = sub { die "receive signal\n" };
    Time::HiRes::ualarm 1000_000;
    sleep 2;
    print "never print";
};          
            
if($@) {    
    print $@;
}           
            
            
eval {      
    local $SIG{ALRM} = sub { die "receive signal\n" };
    Time::HiRes::ualarm 2000_000;
    sleep 1;
    Time::HiRes::ualarm 0; #cancel alarm
};          
            
if($@) {    
    print $@;
}else{                                                                                                                                                                          
    print "cancel alarm signal\n";
}    
```

###tv_interval和gettimeofday实例

+ tv_interval返回两个gettimeofday引用相差的秒数
+ 省略tv_interval的第二个参数，会自动使用当前时间代替

```perl
my @start = Time::HiRes::gettimeofday;
print Dumper(\@start);
sleep(4);
my $diff = Time::HiRes::tv_interval(\@start);
print "$diff\n";
```

###setitimer实例

1. ITIMER_REAL: 真实时间
2. ITIMER_VIRTUAL: 用户时间
3. ITIMER_PROF: 系统时间 

+ getitimer可以取得setitimer还有多久开始执行

```perl
use Time::HiRes qw ( setitimer ITIMER_VIRTUAL time );
$SIG{VTALRM} = sub { print time, "\n" };
setitimer(ITIMER_VIRTUAL, 3.5, 1); 
while(1){ }

```



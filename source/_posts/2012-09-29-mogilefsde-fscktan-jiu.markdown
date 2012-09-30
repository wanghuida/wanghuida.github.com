---
layout: post
title: "mogilefs fsck探究"
date: 2012-09-29 15:13
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---
###fsck的作用
+ 检查mogilefs文件的健康程度，检查的维度有devcount,file_on表,文件长度，校验和（可选）
+ 如果文件丢失，通过路径尝试全设备扫描去查找该文件【只是通过路径去找】

###使用fsck时，碰到的问题:队列保持不变，无法处理后续的文件
```
//启动fsck
mogadm fsck start  
//查看fsck状态，问题出现：进度不变,无法进一步检查文件
mogadm fsck status 
//查看数据库里的队列，发现队列一直保持，没有被处理
select count(*) from file_to_queue where type=1 （type=1代表fsck队列，type=2代表rebalance队列）;
```

###结论（先给出结论，后面有具体的分析流程）
+ 随意取得队列中的某一条记录的devcount等于3，过去操作造成的
+ mindevcount却等于2，这会让fsck进行修复
+ 在修复时，发现3个设备中有一个设备是不可达的，状态也不是dead，返回无法处理，fsck不会把该记录从队列中删除，因为他期望你尽快修复该设备或者设置为dead
+ 解决方法有3个，第一种就是恢复或dead该设备【推荐】，第二种比较山寨,改代码删队列，第三种直接修改数据库,风险较大

###mogadm fsck status详情
```
   Running: 运行状态 
    Status: 完成度
      Time: 执行时间
Check Type: 检查的策略【可以通过fsck_opt_policy_only修改】

NOPA: file_on表里一个都没
POVI: 违反策略
MISS: 在某个设备上没有找到文件
BLEN: file表里的length和磁盘上的文件大小不匹配
GONE: 无法修复了
SRCH: 开始全设备扫描
FOND: 全设备扫描后找了该文件
REPL: 加入复制队列，复制一份文件
BCNT: file表里的devcount不正确
```

<!-- more -->
###MogileFS::Worker::Fsck的work方法分析
+ 因为之前有分析过MogileFS启动源码，所以知道fsck子进程启动就调用MogileFS::Worker::Fsck里的work方法
+ 里面循环调用了最关键的check_fid方法来检查fid对象，返回的结果是已经处理（HANDLED）和没办法处理（STALLED）

```perl
sub work {
    my $self = shift; #perl面向对象获取自己的引用
    POSIX::nice(10);  #调整进程的优先级

    my $sto         = Mgd::get_store(); #获取操作数据库的对象
    
    #2秒调用一次这个匿名方法
    every(2.0, sub {
        my $sleep_set = shift; #一个调整sleep时间的函数，下面可以看到设置为0不去调整，就用2秒
        $nowish = time();      #当前时间
        local $Mgd::nowish = $nowish; #零时变量

        my $queue_todo = $self->queue_todo('fsck');    #获取fsck队列里的数据
        $self->send_to_parent('worker_bored 50 fsck'); #告诉父进程，我还活着
        return unless @{$queue_todo};                  #没有需要需要处理就返回
        return unless $self->validate_dbh;             #验证db，无法连接就退出

        my @fids = ();  #fids数组，用来存放文件的id,也是数据库里文件的唯一标识 
        #遍历队列的数据，$todo就是数据库里的信息 type=1
        while (my $todo = shift @{$queue_todo}) {        
            #new 一个fid对象，只是new，还没有读取fid信息
            my $fid = MogileFS::FID->new($todo->{fid});  
            #这里才是通过查file表，判断他是不是存在
            unless ($fid->exists) {                      
                #从file_to_queue表里删除
                $sto->delete_fid_from_file_to_queue($fid->id, FSCK_QUEUE); 
            }
            push(@fids, $fid); #如果有，那就把fid对象放到数组里
        }
        return unless @fids;   #如果没有需要处理的就返回了 
        #从setting表里读取fsck_opt_policy_only配置
        #1代表只检查数据库信息，0代表检查数据信息和文件长度
        $self->{opt_nostat} = MogileFS::Config->server_setting('fsck_opt_policy_only')     || 0;
        #从setting表里读取fsck_checksum配置,代表是否需要检查校验和,可选值为md5,sha-1,off
        my $alg = MogileFS::Config->server_setting_cached("fsck_checksum");
        if (defined($alg) && $alg eq "off") {   #如果没有定义或为off
            $self->{opt_checksum} = "off";      #那就设置成off,不检查
        } else {
            #验证一下是否为md5 或 sha-1,如果不是那就设置为0
            $self->{opt_checksum} = MogileFS::Checksum->valid_alg($alg) ? $alg : 0;
        }
        MogileFS::FID->mass_load_devids(@fids); #通过file_on表获取每个fid存放在哪些device上
        $sleep_set->(0); #微调sleep时间，写死为0，可能是调试用的

        foreach my $fid (@fids) {               #循环每一个fid对象
            #这里是关键是亮点，是接下去分析的入口
            #返回的结果是已经处理（HANDLED）和没办法处理（STALLED）
            if (!$self->check_fid($fid)) {      
                #如果运行到了这里，说明跟磁盘有连通性的问题
                #最重要的是不会删除队列，放在里面以后再尝试
                #其实队列的卡住的问题就出在这里,接下去进入check_fid看看
                $self->still_alive;
                next;
            }
            #已经处理了，就可以删除队列里的数据了
            $sto->delete_fid_from_file_to_queue($fid->id, FSCK_QUEUE);
        }
    });
}
```

###MogileFS::Worker::Fsck的check_fid方法分析
+ 传入的参数是$fid是MogileFS::FID对象
+ 返回0表示磁盘不可达（STALLED）
+ 返回1代表fid对象已经被处理，不管这个fid是不是有问题，没问题就不处理，有问题就记录日志并修复
+ 真正去修复一个fid对象的函数是fix_fid,接下去会分析
+ 问题渐渐浮出水面，坏掉的设备没有被dead，或者直接删除掉设备有可能会让队列卡住

```perl
#下面是返回值，被定义成常量
use constant STALLED => 0;
use constant HANDLED => 1;
sub check_fid {
    my ($self, $fid) = @_;      #设置对象自己和fid对象

    my $fix = sub {             #定义一个修复的数据的函数,只是定义，真正的调用在下面逻辑
        #fix_fid是最重要的函数用来修复fid对象，也是接下去需要分析的入口
        #这里使用了eval{}，这样可以捕获错误
        my $fixed = eval { $self->fix_fid($fid) }; 
        if (! defined $fixed) {                     #如果产生了错误
            #输出异常日志，我们的问题就会运行到这里输出这样的信息
            error("Fsck stalled for fid $fid: $@"); 
            #返回一个不会去删除队列的结果，造成队列积满
            #所以直接去fix_fid里查看错误的判断就可以知道为什么了
            return STALLED;
        }
        $fid->fsck_log(EV_CANT_FIX) if ! $fixed;  #记录日志，不可以修复,GONE
        $nowish = $self->still_alive;             #更新时间
        return HANDLED;                           #返回已经处理
    };
    
    unless ($fid->devids) {         #一个设备上都没有的话
        $fid->fsck_log(EV_NO_PATHS);#记录日志path都没 NOPA
        return $fix->();            #搜索所有设备作为最后的努力来定位它
    }
    #查找数据库file_on数量，要满足mindevcount，如果超过也属于不满足（神奇的too happy）
    unless ($fid->devids_meet_policy) {     
        $fid->fsck_log(EV_POLICY_VIOLATION);#记录日志违反策略 POVI
        return $fix->();                    #尝试修复
    }

    #如果不skip_devcount，同时file_on和file里的devcount对不起来
    unless (MogileFS::Config->server_setting_cached('skip_devcount') || scalar($fid->devids) == $fid->devcount) {
        $fid->fsck_log(EV_BAD_COUNT); #记录日志file表里devcount不正确
        return $fix->();              #尝试修复，其实是调用$fid->update_devcount();
    }

    #我们不使用，其实他是判断checksum表里是否有记录
    if ($fid->class->hashtype && ! $fid->checksum) { 
        return $fix->();        #尝试修复
    }

    #如果配置了fsck_opt_policy_only那就到此结束了,不会去判断文件长度和校验和了
    #如果策略是这样，事故后的fsck就有意义了
    return HANDLED if $self->{opt_nostat}; 
    #如果配置检查校验和,默认不使用
    if ($self->{opt_checksum} && $self->{opt_checksum} ne "off") { 
        return $fix->(); #尝试修复
    }

    my $err; #记录错误的变量
    #parallel_check_sizes会取得磁盘上文件的大小
    #匿名的对比函数,第一个参数是设备,第二参数就是磁盘上文件的大小
    my $rv = $self->parallel_check_sizes([ $fid->devfids ], sub {
        my ($dfid, $disk_size) = @_;            #devfid对象，和磁盘上文件的大小
        if (! defined $disk_size) {             #如果拿不到磁盘上文件的大小
            my $dev  = $dfid->device;           #设备对象
            if ($dev->dstate->is_perm_dead) {   #如果磁盘的状态是dead
                $err = "needfix";               #记录错误为需要修复
                return 0;                       #返回0
            }
            #输出连通性问题的日志
            error("Connectivity problem reaching device " . $dev->id . " on host " . $dev->host->ip . "\n");
            $err = "stalled";                   #记录错误为无法处理
            return 0;                           #返回0
        }
        return 1 if $disk_size == $fid->length; #file表里的length和磁盘上的大小一样,直接返回1
        $err = "needfix";                       #下面就是大小不一样了，记录错误为需要修复
        return 0;                               #返回0
    });

    if ($rv) {
        return ($fid->class->hashtype && !($self->{opt_checksum} && $self->{opt_checksum} eq "off"))
            ? $fix->() : HANDLED;               #不扯校验和了，直接返回已处理
    } elsif ($err eq "stalled") {               
        #返回无法处理，也会造成队列堆积，因为file_on表里的device没有设置为dead
        return STALLED;
    } elsif ($err eq "needfix") {
        #进行修复
        return $fix->();
    } else {
        #这里应该不会执行到
        die "Unknown error checking fid sizes in parallel.\n";
    }
}
```

###MogileFS::Worker::Fsck的fix_fid方法分析
+ 该方法有可能会比较慢，因为file_on表上一条记录都没会尝试查找所有设备，【通过path】
+ die是由于设备连接性问题造成的，会引起队列堵死

```
use constant CANT_FIX => 0; #不可以被修复
sub fix_fid {
    my ($self, $fid) = @_;                     #对象自己，fid对象
    debug(sprintf("Fixing FID %d", $fid->id)); #调试信息，正在修复哪个fid

    #通过file_on更新一下file表里的devcount; 
    #这里学习了下mysql的get_lock
    $fid->update_devcount;

    #遍历devids属性生成多个devfid对象放到数组,其实就是file_on表
    my @dfids = map { MogileFS::DevFID->new($_, $fid) } $fid->devids; 

    my @good_devs;        #初始化好的设备
    my @bad_devs;         #初始化坏的设备数组
    my %already_checked;  #初始化已经检查过的设备hash

    my $check_dfids = sub { #定义一个监测函数
        my $is_desperate_mode = shift; #是否全设备扫描

        foreach my $dfid (@dfids) { #遍历fid的devfid对象
            my $dev = $dfid->device;
            next if $already_checked{$dev->id}++; #已经检查过就跳过了

            #磁盘状态为dead，那就放到bad_devs里
            if ($dev->dstate->is_perm_dead) {
                push @bad_devs, $dev;
                next;
            }
            
            my $disk_size = $self->size_on_disk($dfid);  #获取磁盘上的文件大小
            #这里是我们产生问题的根源。。。设备不可达，也没设置成dead
            die "dev " . $dev->id . " unreachable" unless defined $disk_size;

            if ($disk_size == $fid->length) { #如果长度一样
                push @good_devs, $dfid->device; #加到好的设备里
                #如果是全设备扫描，那就等于发现了丢失文件，就可以返回了，不用继续查找了
                return if $is_desperate_mode; 
                next; #下一个,其他编程语言的continue
            }

            unless ($is_desperate_mode) { #如果不是全设备扫描
                if ($disk_size == -1) {
                    #没有找到文件呀,MISS
                    $fid->fsck_log(EV_FILE_MISSING, $dev);
                } else {
                    #长度不正确呀,BLEN
                    $fid->fsck_log(EV_BAD_LENGTH, $dev);
                }
            }

            push @bad_devs, $dfid->device; #增加到坏设备里
        }
    };

    $check_dfids->(); #调用上面的监测函数

    unless (@good_devs) { # 如果一个好的都没
        #记录日志,全设备扫描 SRCH
        $fid->fsck_log(EV_START_SEARCH);
        #重新设置@dfids,全部的设备
        @dfids = List::Util::shuffle(
                                     map  { MogileFS::DevFID->new($_, $fid)  }
                                     grep { $_->dstate->should_fsck_search_on }
                                     Mgd::device_factory()->get_all
                                     );
        #重新再找一次,这次是全盘扫描模式
        $check_dfids->("desperate");

        # 不能修复了呀，找不到文件了
        return CANT_FIX unless @good_devs;
        # 如果能找到，记录日志,FOND
        $fid->fsck_log(EV_FOUND_FID);
        # 加到数据库里,就是file_on的表里
        $fid->note_on_device($good_devs[0]); 
    }

    # 循环坏的设备，删除掉file_on里的数据
    foreach my $bdev (@bad_devs) {
        error("removing file_on mapping for fid=" . $fid->id . ", dev=" . $bdev->id);
        $fid->forget_about_device($bdev);
    }

    $fid->want_reload; #重新加载fid对象
    # 如果还是没能满足策略，就就应该复制一份了呀
    unless ($fid->devids_meet_policy) {
        #加入到replacte队列中
        $fid->enqueue_for_replication(in => 1);
        #记录日志 REPL
        $fid->fsck_log(EV_RE_REPLICATE);
        #返回已经处理
        return HANDLED;
    }

    # 更新devcount
    unless(MogileFS::Config->server_setting_cached('skip_devcount') || scalar($fid->devids) == $fid->devcount) {
        $fid->update_devcount(); 
        $fid->fsck_log(EV_BAD_COUNT); #记录日志BCNT
    }

    return HANDLED;
}
```

---
layout: post
title: "MogileFS的replicate进程分析"
date: 2012-10-16 15:47
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

####基本流程
1. 从replicate queue里获取需要处理数据,进行循环
1. 执行真正的【replicate操作】，操作时会锁住这个fid，然后返回结果【下面会对该操作具体分析】 
1. 返回没问题，删除队列并释放锁
1. 返回failed_getting_lock说明正在处理了
1. 返回no_source说明file_on表上没有关于该文件的记录，nexttry设置为最大并释放锁
1. 返回too_happy说明devcount > mindevcount，会尝试删除多余的并释放锁（前提是文件存在，所以不会修复file_on有，文件丢失的情况）
1. 如果以上都不满足，nexttry根据规则增加，failcount递增并释放锁


####replicate操作分析 
+ 简单说replicate操作就是取得合适的device进行复制，合适的意思是尽量避免同一个host的device，没办法才用同一个host的device，有错误就返回具体信息
+ replicate操作才是真正的复制，代码比较多，但是不难理解（难点都会有注释）

<!-- more -->

```perl
sub replicate {
    my ($fid, %opts) = @_;      #参数赋值
    $fid = MogileFS::FID->new($fid) unless ref $fid;   #实例fid对象
    my $fidid = $fid->id; #fid的id

    debug("Replication for $fidid called, opts=".join(',',keys(%opts))) if $Mgd::DEBUG >= 2;

    
    my $errref    = delete $opts{'errref'};  #错误消息的引用
    my $no_unlock = delete $opts{'no_unlock'};  #上锁,使用Mysql的锁,get_lock和release_lock
    #复制源，复制到哪些设备，排除的设备，避免的设备
    my $fixed_source = delete $opts{'source_devid'};
    my $mask_devids  = delete $opts{'mask_devids'}  || {};
    my $avoid_devids = delete $opts{'avoid_devids'} || {};
    my $target_devids = delete $opts{'target_devids'} || []; # inverse of avoid_devids.
    die "unknown_opts" if %opts;
    die unless ref $mask_devids eq "HASH";

    my $sdevid; #下面会看到，就是复制源

    my $sto = Mgd::get_store();
    #释放锁的函数release_lock(name)
    my $unlock = sub {
        $sto->note_done_replicating($fidid);
    };

    #这是一个返回函数，返回结果用的
    my $retunlock = sub {
        my $rv = shift;
        my ($errmsg, $errcode);
        if (@_ == 2) {
            ($errcode, $errmsg) = @_;
            $errmsg = "$errcode: $errmsg"; # include code with message
        } else {
            ($errmsg) = @_;
        }
        $$errref = $errcode if $errref;

        my $ret;
        if ($errcode && $errcode eq "failed_getting_lock") {
            # don't emit a warning with error() on lock failure.  not
            # a big deal, don't scare people.
            $ret = 0;
        } else {
            $ret = $rv ? $rv : error($errmsg);
        }
        if ($no_unlock) {
            die "ERROR: must be called in list context w/ no_unlock" unless wantarray;
            return ($ret, $unlock);
        } else {
            die "ERROR: must not be called in list context w/o no_unlock" if wantarray;
            $unlock->();
            return $ret;
        }
    };

    # hashref of devid -> MogileFS::Device所有的device
    my $devs = Mgd::device_factory()->map_by_id
        or die "No device map";

    #错误话就调用retunlock 
    return $retunlock->(0, "failed_getting_lock", "Unable to obtain lock for fid $fidid")
        unless $sto->should_begin_replicating_fidid($fidid);#这里获取mysql的锁 get_lock(name,timeout),name里fidid

    # if the fid doesn't even exist, consider our job done!  no point
    # replicating file contents of a file no longer in the namespace.
    #没有fid返回
    return $retunlock->("nofid") unless $fid->exists;

    #Class.pm对象
    my $cls = $fid->class;
    #策略对象MogileFS::ReplicationPolicy::MultipleHosts
    my $polobj = $cls->repl_policy_obj;

    # learn what this devices file is already on
    #所有设备
    my @on_devs;         # all devices fid is on, reachable or not.
    #排除dead和drain的设备
    my @on_devs_tellpol; # subset of @on_devs, to tell the policy class about
    #可读的设备
    my @on_up_devid;     # subset of @on_devs:  just devs that are readable
    #设置上面的数组
    foreach my $devid ($fid->devids) {
        my $d = Mgd::device_factory()->get_by_id($devid)
            or next;
        #fid在哪些dev上
        push @on_devs, $d;
        #should_hava_files表示状态不为drain或者dead
        if ($d->dstate->should_have_files && ! $mask_devids->{$devid}) {
            push @on_devs_tellpol, $d;
        }
        #有read权限
        if ($d->dstate->can_read_from) {
            push @on_up_devid, $devid;
        }
    }
    #如果一个设备都没，就返回
    return $retunlock->(0, "no_source",   "Source is no longer available replicating $fidid") if @on_devs == 0;
    return $retunlock->(0, "source_down", "No alive devices available replicating $fidid") if @on_up_devid == 0;

    #fixed_source设置错误，根本就没有
    if ($fixed_source && ! grep { $_ == $fixed_source } @on_up_devid) {
        error("Fixed source dev$fixed_source requested for $fidid but not available. Trying other devices");
    }

    #错误的设备记录，下面循环里就不会再使用了
    my %dest_failed;    # devid -> 1 for each devid we were asked to copy to, but failed.
    my %source_failed;  # devid -> 1 for each devid we had problems reading from.
    #用来标记真的复制了，下面的循环会使用
    my $got_copy_request = 0;  # true once replication policy asks us to move something somewhere
    my $copy_err;

    #过滤复制到哪些设备
    my $dest_devs = $devs;
    if (@$target_devids) {
        $dest_devs = {map { $_ => $devs->{$_} } @$target_devids};
    }

    my $rr;  #MogileFS::ReplicationRequest对象
    #开始循环复制，因为会有可能要复制3份的嘛。
    while (1) {
        #MogileFS::ReplicationPolicy::MultipleHosts的replicate_to
        #取得ReplicationRequest对象【该对象是通过ReplicationPolicy::MultipleHosts策略生成的】
        #该对象会有4种返回值，分别是，1：对象形式，说明需要并可以复制。2：tmp_no_answer，没有可选择的device。
        #3：all_good，一切OK。4：too_good，好过头了呀。
        $rr = rr_upgrade($polobj->replicate_to(
                                               fid       => $fidid, #fidid
                                               on_devs   => \@on_devs_tellpol, # all device objects fid is on, dead or otherwise
                                               all_devs  => $dest_devs, #所有的dev，如果有target_devids的话那就用target的咯
                                               failed    => \%dest_failed, #失败信息
                                               min       => $cls->mindevcount, #最小设备
                                               ));
        #是happy就跳出，跳出会有后续处理的
        last if $rr->is_happy;

        #经过过滤的dev
        my @ddevs;  # dest devs, in order of preference
        #选择出来的devid
        my $ddevid; # dest devid we've chosen to copy to
        #这里设置ddevs,其实就是host不同的设备
        if (@ddevs = $rr->copy_to_one_of_ideally) {
            #过滤掉不希望的
            if (my @not_masked_ids = (grep { ! $mask_devids->{$_} &&
                                             ! $avoid_devids->{$_}
                                         }
                                      map { $_->id } @ddevs)) {
                $ddevid = $not_masked_ids[0]; #拿一个放到ddevid
            } else {
                return $retunlock->("would_worsen");
            }
        } elsif (@ddevs = $rr->copy_to_one_of_desperate) { #没有不同的host，没办法同一个host也行啊
            # TODO: reschedule a replication for 'n' minutes in future, or
            # when new hosts/devices become available or change state
            $ddevid = $ddevs[0]->id;
        } else {
            #跳出了呀，
            last;
        }
        #标记一下真的复制了
        $got_copy_request = 1;

        # replication policy shouldn't tell us to put a file on a device
        # we've already told it that we've failed at.  so if we get that response,
        # the policy plugin is broken and we should terminate now.
        #不在错误数组里
        if ($dest_failed{$ddevid}) {
            return $retunlock->(0, "policy_error_doing_failed",
                                "replication policy told us to do something we already told it we failed at while replicating fid $fidid");
        }

        # replication policy shouldn't tell us to put a file on a
        # device that it's already on.  that's just stupid.
        #已经在这file_on上了么，不行的呀
        if (grep { $_->id == $ddevid } @on_devs) {
            return $retunlock->(0, "policy_error_already_there",
                                "replication policy told us to put fid $fidid on dev $ddevid, but it's already there!");
        }

        # find where we're replicating from
        #这里开始设置复制源
        {
            # TODO: use an observed good device+host as source to start.
            #排除掉错误源
            my @choices = grep { ! $source_failed{$_} } @on_up_devid;
            return $retunlock->(0, "source_down", "No devices available replicating $fidid") unless @choices;
            #如果有fixed_source那就用，没有么就随即选一个吧
            if ($fixed_source && grep { $_ == $fixed_source } @choices) {
                $sdevid = $fixed_source;
            } else {
                @choices = List::Util::shuffle(@choices);
                MogileFS::run_global_hook('replicate_order_final_choices', $devs, \@choices);
                $sdevid = shift @choices;
            }
        }

        #是子进程的判断
        my $worker = MogileFS::ProcManager->is_child or die;
        #校验和，不用的呀，不考虑
        my $digest = Digest->new($cls->hashname) if $cls->hashtype;
        #真的复制了
        my $rv = http_copy(
                           sdevid       => $sdevid,
                           ddevid       => $ddevid,
                           fid          => $fidid,
                           rfid         => $fid,
                           expected_len => undef,  # FIXME: get this info to pass along
                           errref       => \$copy_err,
                           callback     => sub { $worker->still_alive; },
                           digest       => $digest,
                           );
        die "Bogus error code: $copy_err" if !$rv && $copy_err !~ /^(?:src|dest)_error$/;
        #如果错误了，设置失败数组，next等于其他语言的continue呀，继续找其他的
        unless ($rv) {
            error("Failed copying fid $fidid from devid $sdevid to devid $ddevid (error type: $copy_err)");
            if ($copy_err eq "src_error") {
                $source_failed{$sdevid} = 1;

                if ($fixed_source && $fixed_source == $sdevid) {
                    error("Fixed source dev$fixed_source was requested for $fidid but failed: will try other sources");
                }

            } else {
                $dest_failed{$ddevid} = 1;
            }
            next;
        }
        #复制成功了，开始插入数据了
        my $dfid = MogileFS::DevFID->new($ddevid, $fid);
        $dfid->add_to_db;
        if ($digest && !$fid->checksum) {
            $sto->set_checksum($fidid, $cls->hashtype, $digest->digest);
        }

        #设置一下变量，加入要复制3分是应该再继续复制的
        push @on_devs, $devs->{$ddevid};
        push @on_devs_tellpol, $devs->{$ddevid};
        push @on_up_devid, $ddevid;
    }

    # We are over replicated. Let caller decide if it should rebalance.
    #好过头了
    if ($rr->too_happy) {
        return $retunlock->(0, "too_happy", "fid $fidid is on too many devices");
    }
    #这是满足策略的情况
    if ($rr->is_happy) {
        return $retunlock->(1) if $got_copy_request;
        #没有地方去复制了
        return $retunlock->("lost_race");  # some other process got to it first.  policy was happy immediately.
    }
    #应该不会走到这里了
    return $retunlock->(0, "policy_no_suggestions",
                        "replication policy ran out of suggestions for us replicating fid $fidid");
}
```

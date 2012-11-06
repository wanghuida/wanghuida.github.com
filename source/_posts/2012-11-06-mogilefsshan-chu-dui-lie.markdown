---
layout: post
title: "mogilefs删除队列"
date: 2012-11-06 19:31
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

###遇到的问题

+ 发现file_to_delete2队列表里有500万记录，奇怪这些文件为什么要删除？心里不踏实，寻找根源...
+ 在队列中随便拿一个fid，在file表里查竟然是空的，再找file_on表竟然有数据...


###什么时候会插入fid到删除队列呢
1. cmd_create_close时，如果连接终端，文件大小等不正确，会增加到删除队列  
1. cmd_create_close时，如果同一domain下key已经存在，那么会去删除原有的文件(两份文件key相同，但是fid不同，我的问题就出在这)
1. mogdelete工具会删除memcache和file表的文件记录，并运用删除队列删除磁盘上的文件和file_on表
1. client调用delete方法，和mogdelete一样

<!-- more -->

###cmd_create_close逻辑

+ mogilefs上传文件的逻辑是先问tracker拿文件存储路径，然后上传文件到一个storage，最后调用cmd_create_close方法插入数据记录和replicate队列

```perl
sub cmd_create_close {                                                                                                                                                          
    my MogileFS::Worker::Query $self = shift;
    my $args = shift;
      
    # has to be filled out for some plugins
    $args->{dmid} = $self->check_domain($args)
        or return $self->err_line('domain_not_found');
      
    # call out to a hook that might modify the arguments for us
    # 调用插件用的钩子，一般是没的
    MogileFS::run_global_hook('cmd_create_close', $args);
      
    # late validation of parameters
    # 设置下面使用的变量
    my $dmid  = $args->{dmid};
    my $key   = $args->{key};
    my $fidid = $args->{fid}    or return $self->err_line("no_fid");
    my $devid = $args->{devid}  or return $self->err_line("no_devid");
    my $path  = $args->{path}   or return $self->err_line("no_path");
    my $checksum = $args->{checksum};
      
    if ($checksum) {
        $checksum = eval { MogileFS::Checksum->from_string($fidid, $checksum) };
        return $self->err_line("invalid_checksum_format") if $@;
    }    
      
    # 初始化fid和dfid对象
    my $fid  = MogileFS::FID->new($fidid);
    my $dfid = MogileFS::DevFID->new($devid, $fid);
      
    # is the provided path what we'd expect for this fid/devid?
    # 验证是否为伪造路径
    return $self->err_line("bogus_args")
        unless $path eq $dfid->url;
    my $sto = Mgd::get_store();
     
    # find the temp file we're closing and making real.  If another worker
    # already has it, bail out---the client closed it twice.
    # this is racy, but the only expected use case is a client retrying.
    # should still be fixed better once more scalable locking is available.
    # 取得零时文件数据,并删除掉
    my $trow = $sto->delete_and_return_tempfile_row($fidid) or
        return $self->err_line("no_temp_file");
     
    # Protect against leaving orphaned uploads.
    # 定义错误函数，可以看到$fid->delete会删除file表记录并插入删除队列
    my $failed = sub {
        $dfid->add_to_db;
        $fid->delete;
    };
     
    # 验证设备
    unless ($trow->{devids} =~ m/\b$devid\b/) {
        $failed->();
        return $self->err_line("invalid_destdev", "File uploaded to invalid dest $devid. Valid devices were: " . $trow->{devids});
    }
     
    # if a temp file is closed without a provided-key, that means to
    # delete it.
    # 验证key
    unless (defined $key && length($key)) {
        $failed->();
        return $self->ok_line;
    }
     
    # get size of file and verify that it matches what we were given, if anything
    my $httpfile = MogileFS::HTTPFile->at($path);
    my $size = $httpfile->size;

    # size check is optional? Needs to support zero byte files.
    # 验证文件大小
    $args->{size} = -1 unless $args->{size};
    if (!defined($size) || $size == MogileFS::HTTPFile::FILE_MISSING) {
        # storage node is unreachable or the file is missing
        my $type    = defined $size ? "missing" : "cantreach";
        my $lasterr = MogileFS::Util::last_error();
        $failed->();
        return $self->err_line("size_verify_error", "Expected: $args->{size}; actual: 0 ($type); path: $path; error: $lasterr")
    }
     
    if ($args->{size} > -1 && ($args->{size} != $size)) {
        $failed->();
        return $self->err_line("size_mismatch", "Expected: $args->{size}; actual: $size; path: $path")
    }
     
    # checksum validation is optional as it can be very expensive
    # However, we /always/ verify it if the client wants us to, even
    # if the class does not enforce or store it.
    # 校验和
    if ($checksum && $args->{checksumverify}) {
        my $alg = $checksum->hashname;
        my $actual = $httpfile->digest($alg, sub { $self->still_alive });
        if ($actual ne $checksum->{checksum}) {
            $failed->();
            $actual = "$alg:" . unpack("H*", $actual);
            return $self->err_line("checksum_mismatch",
                           "Expected: $checksum; actual: $actual; path: $path");
        }
    }
    # see if we have a fid for this key already
    # 关键的来了，如果在同一个domain下有相同的key，那么删除该文件的file表记录，并插入delete队列
    my $old_fid = MogileFS::FID->new_from_dmid_and_key($dmid, $key);
    if ($old_fid) {
        # Fail if a file already exists for this fid.  Should never
        # happen, as it should not be possible to close a file twice.
        return $self->err_line("fid_exists")
            unless $old_fid->{fidid} != $fidid;
     
        $old_fid->delete;
    }
     
    # TODO: check for EIO?
     
    # insert file_on row
    # 以上都没有问题，那么就确认保存文件到数据库
    # 保存到file_on表
    $dfid->add_to_db;
     
    $checksum->maybe_save($dmid, $trow->{classid}) if $checksum;
    
    # 保存到file表 
    $sto->replace_into_file(
                            fidid   => $fidid,
                            dmid    => $dmid,
                            key     => $key,
                            length  => $size,
                            classid => $trow->{classid},
                            devcount => 1,
                            );
     
    # mark it as needing replicating:
    # 插入文件到replicate队列 
    $fid->enqueue_for_replication();
     
    # call the hook - if this fails, we need to back the file out
    # 调用钩子
    my $rv = MogileFS::run_global_hook('file_stored', $args);
    if (defined $rv && ! $rv) { # undef = no hooks, 1 = success, 0 = failure
        $fid->delete;                                                                                                                                                           
        return $self->err_line("plugin_aborted");
    }
    # all went well, we would've hit condthrow on DB errors
    # 返回正确结果
    return $self->ok_line;
}    
```

###总结为什么会出现这种现象

+ 当在同一个domain下，上传相同key的文件，之前的文件会被删除，但是两个文件的fid是不同的
+ $fid->delete这个操作实际是删除memcache和file表数据，file_on表和磁盘上的文件交给队列来处理
+ 作者考虑到删除文件是比较耗时的操作，所以交给队列慢慢处理，才会导致file表里没数据，但是file_on上仍有数据
+ 如果可能，尽量避免上传相同的key，这样可以减少对db的读写操作(我准备优化)


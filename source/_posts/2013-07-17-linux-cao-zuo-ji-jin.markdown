---
layout: post
title: "linux 操作集锦"
date: 2013-07-17 19:18
comments: true
sharing: true
footer: true
categories: [Share]
---

## 第一部
###1. 以Sudo运行上条命令       
    $ sudo !!

通常出现的情况是，敲完命令执行后报错才发现忘了`sudo`。这时，新手用户就会`:`按上箭头，按左箭头，盯着光标回到开始处，输入`sudo`，回车;高手用户就蛋定多了，按`Ctrl-p`，按`Ctrl-a`，输入`sudo`，回车。这里介绍这个是天外飞仙级别的，对，就直接`sudo !!`。

两个叹号是bash的一个特性，称为事件引用符(event designators)。`!!`相当于`!-1`，引用前一条命令，当然也可以`!-2`，`!-50`。默认情况下bash会在`~/.bash_history`文件内记录用户执行的最近500条命令，`history`命令可以显示这些命令.

<!-- more -->

###2. 以HTTP方式共享当前文件夹的文件
    $ python -m SimpleHTTPServer

这命令启动了Python的SimpleHTTPServer模块，命令执行后将在本机8000端口开放HTTP服务，在其他能访问本机的机器的浏览器打开http://ip:8000即打开一个目录列表，点击即可下载。

额外的:用 python 快速开启一个`SMTP`服务

    $ python -m smtpd -n -c DebuggingServer localhost:1025

这是一个用Python标准库smtpd(用`-m smtpd`指定)实现在简易SMTP服务，运行于1025端口 。另外三个参数的解释:

    -n    参数让Python不要进行setuid(改变用户)为"nobody"，也就是说直接用你的帐号来运行
    -c DebuggingServer    让Python运行时在屏幕上输出调试及运行信息
    localhost:1025        让Python在本地的1025端口上开启SMTP服务

另外，假如想让程序运行于标准的25的端口上的话，必须使用`sudo`命令，因为只有root才能在1-1024端口上开启服务。如下:

    sudo python -m smtpd -n -c DebuggingServer localhost:25

###3. 在以普通用户打开的VIM当中保存一个ROOT用户文件
    :w !sudo tee %

常常忘记了`sudo`就直接用vim编辑/etc内的文件，等编辑好了，保存时候才发现没权限。查阅vim的文档(输入`:help :w`)，会提到命令`:w!{cmd}`，让vim执行一个外部命令{cmd}，然后把当前缓冲区的内容从stdin传入。tee是一个把stdin保存到文件的小工具。而%，是vim当中一个只读寄存器的名字，总保存着当前编辑文件的文件路径。所以执行这个命令，就相当于从vim外部修改了当前编辑的文件。

###4. 切换回上一个目录
    $ cd -

横杆-代表上一个目录的路径。实际上`cd -`就是`cd $OLDPWD`的简写，bash的固定变量`$OLDPWD`总保存着之前一个目录的路径。相对地，`$PWD`总保存着当前目录的路径。这些变量在编写shell脚本时候相当有用。

###5. 替换上一条命令中的一个短语
    $ ^foo^bar^
　　
又是另外一个事件引用符，可以把上一条命令当中的foo替换成bar。在需要重复运行调试一道长长的命令，需要测试某个参数时候，用这个命令会比较实用。这道命令的原始样式应该是这样的:

    !!:s/foo/bar/

本文一开始介绍过`!!`，vim、sed的替换操作都是这样的语法。
###6. 快速备份一个文件
    $ cp filename{，.bak}

这道命令把filename文件拷贝成filename.bak，在一些比较复杂的安装教程里面可以见到这样的用法。其原理就在于bash对大括号的展开操作，`filename{，.bak}`这一段会被展开成`filename filename.bak`再传给cp，于是就有了备份的命令了。大括号在bash里面是一个排列的意义，`$ echo {a，b，c}{a，b，c}{a，b，c}`将输出三个集合的全排列

###7. 免密码SSH登录主机
    $ ssh-copy-id remote-machine

这个命令把当前用户的公钥串写入到远程主机的`~/.ssh/authorized_keys`内，这样下次使用ssh登录的时候，远程主机就直接根据这串密钥完成身份校验，不再询问密码。前提是你当前用户已经生成了公钥，因为默认是没有的，先执行`ssh-keygen`试试吧!

这个命令如果用手工完成，是这样的:

    your-machine$ scp ~/.ssh/identity.pub remote-machine:
    your-machine$ ssh remote-machine
    remote-machine$ cat identity.pub >> ~/.ssh/authorized_keys
 
如果你想删掉远程主机上的密钥，直接打开authorized_keys，搜索你的用户名，删除那行即可。

###8. 抓取LINUX桌面的视频
    $ ffmpeg -f x11grab -s wxga -r 25 -i :0.0 -sameq /tmp/out.mpg

我们在一些视频网站上看到别人的3D桌面怎么怎么酷的视频，通常就是这么来的，ffmpeg可以直接解码X11的图形，并转换到相应输出格式。ffmpeg的通常用法是，根据一堆参数，输出一个文件，输出文件通常放最后，下面解析下几个参数:

    -f x11grab  指定输入类型。因为x11的缓冲区不是普通的视频文件可以侦测格式，必须指定后ffmpeg才知道如何获得输入
    -s wxga 设置抓取区域的大小。wxga是`1366*768`的标准说法，也可以换成`-s 800×600`的写法
    -r 25   设置帧率，即每秒抓取的画面数
    -i :0.0 设置输入源，本地X默认在0.0
    -sameq   保持跟输入流一样的图像质量，以用来后期处理

## 第二部分
###1. 用你最喜欢的编辑器来敲命令
    command <CTRL-x CTRL-e>

在已经敲完的命令后按`<CTRL-x CTRL-e>`，会打开一个指定的编辑器(比如vim，通过环境变量`$EDITOR`指定)，里面就是你刚输入的命令，然后进行编辑，特别是对那些参数异常复杂的程序，比如`mencoder/ffmpeg`，一个命令动辄3、4行的。

实际上这是readline库的功能，在默认情况下，bash使用的是emacs模式的命令行操作方式， `<CTRL-x CTRL-e>`是调用这个功能的一个绑定。如果你习惯使用vi模式，按`<ESC v>`可以实现同样功能。如果喜欢别的编辑器，可以在`~/.bashrc`里面放上比如`export EDITOR=nano`的命令。

另外一个修改命令的方法是使用`fc`命令(Fix Command)，在编辑器里面打开上一句命令。我们的第一辑连载提过一个`^foo^bar^`命令可以用fc来实现:`fc -s foo=bar`。

###2. 清空或创建一个文件
    > file.txt

`>`在shell里面是标准输出重定向符，即把命令行输出转往一个文件内，但这里没有”前部命令”，输出为空，于是就覆盖(或创建)成一个空文件了。有些脚本的写法是`:>file.txt`，因为`:`是bash默认存在的空函数。单纯创建文件也可以用`$touch file.txt`，touch本来是用作修改文件的时间戳，但如果文件不存在，就自动创建了。

###3. 用SSH创建端口转发通道
    ssh -N -L2001:remotehost:80 user@somemachine

这个命令在本机打开了2001端口，对本机2001端口的请求通过somemachine作为跳板，转到remotehost的80端口上。实现效果跟术语反向代理是相似的，实际上就是端口转发，注意上面的描述涉及了3台主机，但当然somemachine可以变成localhost。这个命令比较抽象，但有时候是很有用的，比如因为众所周知的原因国内的IP的80端口无法使用，又或者公司的防火墙只给外网开了ssh端口，需要访问内部服务器一个web应用，以及需要访问某些限定了来源IP的服务，就可以用上这个方法了。举一个具体例子，运行:

    ssh -f -N -L 0.0.0.0:443:twitter.com:443 shell.cjb.net
    ssh -f -N -L 0.0.0.0:80:twitter.com:80 shell.cjb.net

然后在`/etc/hosts`里面添加`127.0.0.1 twitter.com`，剩下的你懂的。

###4. 重置终端
    reset

如果你不小心`cat`了某个二进制文件，很可能整个终端就傻掉了，可能不会换行，没法回显，大堆乱码之类的，这时候敲入`reset`回车，不管命令有没有显示，就能回复正常了。实际上`reset`命令只是输出了一些特殊字符，BusyBox里面最简单的reset程序的实现:

    printf(“\033c\033(K\033[J\033[0m\033[?25h”);

输出的这些字符对Shell是有特殊意义的:   

    \033c   ESC c   发送重置命令
    \033(K  ESC ( K 重载终端的字符映射
    \033[J  ESC [ J 清空终端内容
    \033[0m ESC [ 0 m   初始化字符显示属性
    \033[?25h:  ESC [ ? 25 h    让光标可见

其中字符显示属性经常用来设定打印字符的颜色等

###5. 在午夜的时候执行某命令
    echo cmd | at midnight

说的就是`at`这个组件，通常跟`cron`相提并论，不过`at`主要用于定时一次性任务，而`cron`定时周期性任务。`at`的参数比较人性化，跟英语语法一样，可以`tomorrow`，`next week`之类的。

###6. 远程传送麦克风语音（实现一个喊话器）
    dd if=/dev/dsp | ssh username@host dd of=/dev/dsp

`/dev/dsp`是Linux下声卡的文件映射(Digital Signal Proccessor)，从其中读数据就是录音，往里面写数据就是播放，相当简单!dd是常用的数据拷贝程序，如果不同时指定`if`、`of`，就直接使用`stdin/stdout`来传输。如果没有远程主机，可以试试这样:

    dd if=/dev/dsp of=/dev/dsp

直接回放麦克风的声音，只是有一点延时。但是如果有别的程序正在使用声卡，这个方法就不凑效了，因为一般的声卡都不允许多个音频流同时处理，可以借用alsa组件的工具，arecord跟aplay:

    arecord | ssh username@host aplay

本地回放:

    arecord | aplay

如果你想吓吓别人:

    cat /dev/urandom | ssh username@host aplay

###7. 映射一个内存目录
    mount -t tmpfs -o size=1024m tmpfs /mnt/ram

这个命令开了一块1G内存来当目录用。不过放心，如果里面没文件，是不会占用内存的，用多少占多少。一般来说没必要手动挂载，因为多数发行版都会在fstab内预留了一个内存目录，挂载在`/dev/shm`，直接使用即可;最常见的用途是用内存空间来放Firefox的配置，可以让慢吞吞的FF快很多。

###8. 用DIFF对比远程文件跟本地文件
    ssh user@host cat /path/to/remotefile | diff /path/to/localfile -

`diff`通常的用法是从参数读入两个文件，而命令里面的-则是指从stdin读入了。善用ssh可以让web开发减少很多繁琐，还有比如sshfs，可以从编辑-上传-编辑-上传的人工循环里面解脱出来。

###9. 查看系统中占用端口的进程
    netstat -tulnp

Netstat是很常用的用来查看Linux网络系统的工具之一，这个参数可以背下来:

    -t  显示TCP链接信息
    -u  显示UDP链接信息
    -l  显示监听状态的端口
    -n  直接显示ip，不做名称转换
    -p  显示相应的进程PID以及名称(要root权限)

如果要查看关于sockets更详细占用信息等，可以使用lsof工具。

##第三部分
###1. 更友好的显示当前挂载的文件系统
    mount | column -t

这条命令适用于任何文件系统，`column`用于把输出结果进行列表格式化操作，这里最主要的目的是熟悉`columnt`的用法。另外可加上列名称来改善输出结果：

    (echo "DEVICE - PATH - TYPE FLAGS" && mount) | column -t

列2和列4并不是很友好，可以用awk来再处理一下:

    (echo "DEVICE PATH TYPE FLAGS" && mount|awk '$2=$4="";1')|column -t
最后可以设置一个别名，为nicemount

    nicemount(){
    　　(echo "DEVICE PATH TYPE FLAGS" && mount|awk '$2=$4="";1')|column -t; 
    }

直接输入nicemount运行即可。

###2. 运行前一个Shell命令，同时用"bar"替换掉命令行中的每一个"foo"
    !!:gs/foo/bar

`!!`表示重复执行上一条命令，并用`:gs/foo/bar`进行替换操作。
###3. 实时某个目录下查看最新改动过的文件
    watch -d -n 1 'df; ls -FlAt /path'

在使用这条命令时你需要替换其中的/path部分，`watch`是实时监控工具，`-d`参数会高亮显示变化的区域，`-n 1`参数表示刷新间隔为1秒。`df; ls -FlAt /path`运行了两条命令，`df`是输出磁盘使用情况，`ls -FlAt`则列出/path下面的所有文件。`ls -FlAt`的参数详解:

    -F  在文件后面加一个文件符号表示文件类型，共有 */=>@| 这几种类型，* 表示可执行文件，/ 表示目录，= 表示接口( sockets) ，> 表示门， @ 表示符号链接， | 表示管道
    -l  以列表方式显示
    -A  显示 . 和 ..
    -t  根据时间排序文件

###4. 通过SSH挂载远程主机上的文件夹
    sshfs name@server:/path/to/folder /path/to/mount/point

这条命令可以让你通过 SSH 加载远程主机上的文件系统为本地磁盘，前提是你需要安装FUSE及sshfs这两个软件。卸载的话使用`fusermount`命令:

    fusermount -u /path/to/mount/point

###5. 用 Wget 的递归方式下载整个网站
    wget --random-wait -r -p -e robots=off -U Mozilla www.example.com

    - -random-wait  等待0.5到1.5秒的时间来进行下一次请求
    -r  开启递归检索
    -e robots=off   忽略robots.txt
    -U Mozilla  设置 User-Agent 头为 Mozilla

###6. 执行一条命令但不保存到 history 中
    <space> command

这条命令可运行于最新的Bash shell里，在其它shell中没测试过。通过在命令行前面添加一个空格，就可以阻止这条命令被保存到`~/.bash_history`文件中，这个行为可以通过`$HISTIGNORE`shell变量来控制。设置为`HISTIGNORE="&:[ ]*"`，表示不保存重复的命令到`history`中，并且不保存以空格开头的命令行。`$HISTIGNOR`中的值以冒号分隔。

###7. 显示当前目录中所有子目录的大小
    du -h --max-depth=1

`--max-depth=1`参数可以让du命令显示当前目录下1级子目录的统计信息，当然你也可以把1改为2，进一步显示2级子目录的统计信息，可以灵活运用。而`-h`参数则是以Mb 、G这样的单位来显示大小。小工具ncdu可以更方便的达到此效果。

###8. 显示消耗内存最多的10个运行中的进程，以内存使用量排序
    ps aux | sort -nk +4 | tail

这是一个典型的管道应用，通过`ps aux`来输出到`sort`命令，并用`sort`排序列出4栏，再进一步转到`tail`命令，最终输出10行显示使用内存最多的进程情况。假如想要发现哪个进程使用了大量内存的话，通常会使用`hto`p或`top`而非`ps`。

##第四部分
###1. 查看ASCII码表
    man 7 ascii

上述命令就能很详细的方式解释ascii编码，当然这里还有在线版。man命令的第二个参数是区域码，用来区分索引词的范围，比如printf， C标准库里面的printf跟bash当中的printf是不同的，前者的查询是`man 3 printf`，后者是`man 1 printf`。如果这个区域码省略，就会从1开始搜索，直到找到为止。命令`man man`可以看到详细的解释。

manpages里面还有一些有趣而且实用的资料，可能鲜为人知:

    man 1 intro 一篇对从未接触过Linux的用户的简明教程
    man 2 syscalls  内核系统请求的列表，按内核版本注释分类，系统编程必备
    man 2 select_tut    关于select()系统请求的教程
    man 3 string    在头文件内的所有函数
    man 3 stdio 关于头文件的使用，标准输入/输出库的说明
    man 4 console_codes Linux的终端控制码及其使用解释
    man 4 full  介绍/dev/full这个总是处于"满"状态的磁盘。(对应/dev/null这个总是空的设备)
    man 5 proc  介绍/proc下的文件系统
    man 5 filesystems   各种Linux文件系统
    man 7 bootparam 详细解释内核启动参数
    man 7 charsets  解释各种语言的编码集。(gbk，gb2312等)
    man 7 glob  解释glob文件名管理机制的工作过程
    man 7 hier  解释Linux文件系统结构各个部分的作用
    man 7 operator  C语言的运算符的列表
    man 7 regex 介绍正则表达式

###2. 简易计时器
    time read

运行命令开始算起，到结束时按一Enter，就显示出整个过程的时间，精确到ms级别。time是用来计算一个进程在运行到结束过程耗费多少时间的程序，它的输出通常有三项:

    $ time ls /opt
    ...
    real 0m0.008s
    user 0m0.003s
    sys 0m0.007s

    real    指整个程序对真实世界而言运行所需时间
    user    指程序在用户空间运行的时间，
    sys     指程序对系统调用锁占用时间

`read`本来是一个读取用户输入的命令，常见用法是`read LINE`，用户输入并回车后，键入的内容就被保存到`$LINE`变量内，但在键入回车前，这个命令是一直阻塞的。可见`time read`这命令灵活地利用了操作系统的阻塞。

###3. 在一个子SHELL中运行一个命令
    (cd /tmp && ls)

当然这只是演示，要查看目录当然可以`ls /tmp`。好处就是不会改变当前shell的目录，以及如果命令中设计环境变量，也不会对当前shell有任何修改。

在Shell编程中还有很多使用上引号来括住一个命令:`ls /tmp`，这也是子shell过程。可是上引号的方法无法嵌套，而使用小括号的方法可以，一个比较纠结的例子是:

    echo $(echo -e file:///C|/x%24%28printf "%x" 65))

###4. 利用中间管道嵌套使用SSH
    ssh -t host_A ssh host_B

如果目标机器host_B处于比较复杂的网络环境，本机无法直接访问，但另外一台host_A能够访问到host_B，而且也能被本机访问到，那上述命令就解决了方便登录host_B的问题。但理论上这个过程是可以无限嵌套的，比如:

    ssh -t host1 ssh -t host2 ssh -t host3 ssh -t host4 ...

###5. 清空屏幕
    <CTRL+l>

这个跟之前介绍的`reset`命令重置终端的作用有些类似，其实都只是发送一段控制序列，让终端的显示复位。还可以这样运行:

    tput clear

`tput`是专门用来控制终端的一个小工具，也挺强大的，详细信息运行`man tput`查看。
###6. 我想知道一台服务器什么时候重启完
    ping -a IP

`ping`命令有个audible ping参数`-a`，当ping通服务器时会让小喇叭叫起来。

###7. 列出你最常用的10条命令
    history | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head

这行命令组合得很妙: `history`输出用户了命令历史;`awk`统计并输出列表;`sort`排序;`head`截出前10行。

###8. 用TELNET看《星球大战》
    telnet towel.blinkenlights.nl

没什么好解释的，就是ASCII艺术之一。

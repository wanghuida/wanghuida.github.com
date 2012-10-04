---
layout: post
title: "Aho-Corasick算法"
date: 2012-09-27 14:10
comments: true
sharing: true
footer: true
categories: [Share, Perl]
---

###aho-corasick算法优势
+ 查找时无需回溯，性能相当好
+ 当需要查找或替换的词非常多时，生成的trie树可以方便缓存

###学习方法
+ 了解ac算法的基本思想
+ 阅读源码（看过php, java, perl的版本，还最推荐perl, 因为perl语法精炼,易于阅读）
+ ac模拟器（见下面的flash，可以尝试操作一下）

<!-- more -->

<object width="650" height="500" type="application/x-shockwave-flash" data="http://www.ivank.net/blogspot/en/AHODrawing.swf">
<param value="http://www.ivank.net/blogspot/en/AHODrawing.swf" name="movie">
<!-- http://blog.ivank.net/aho-corasick-algorithm-in-as3.html -->
</object>

###测试Perl的aho-corasick搜索

```perl
use Algorithm::AhoCorasick::SearchMachine;

use strict;
use Data::Dumper;
use utf8;

my $text = "王惠达啊"; #被搜索的内容
my @keywords = ("王惠达","惠达啊"); #需要查找的关键字,这两个关键字是设计过的
my $machine = Algorithm::AhoCorasick::SearchMachine->new(@keywords); #实例化

my %total;
my $handle_all = sub { #回调函数，把位置和关键字收集起来
    my ($pos, $keyword) = @_; 
    if (!exists($total{$pos})) {
        $total{$pos} = [ ];
    }   
    push @{$total{$pos}}, $keyword;
    undef;
};

$machine->feed($text, $handle_all); #查找

print Dumper(\%total); #打印结果
```

###aho-corasick建立树过程
+ 将关键字建立起一颗Tire树
+ 设置第一层的不匹配的转移节点
+ 设置其余的不匹配转移节点
+ 初始化状态到根节点

```
sub _build_tree {
    my $self = shift;

    #建立根节点，Node对象包含属性parent,char,results,transitions,failure
    #parent属性放置的也是Node对象
    #char属性是一个字符
    #results属性代表匹配的串,当达到这个状态时可以直接返回
    #transitions属性用来描述树型结构的hash,以char为key,Node对象为value
    #failure属性放置的也是Node对象，当匹配失败时，转移到哪个状态
    $self->{root} = Algorithm::AhoCorasick::Node->new();

    #将关键字建立起一颗树
    foreach my $p (@{$self->{keywords}}) { #循环传进来的keywords数组
        my $nd = $self->{root}; #每次都是从根节点开始的
        foreach my $c (split //, $p) { #切割成字符
            my $ndNew = $nd->get_transition($c); #如果父节点已经包含该字符,直接取得该节点
            if (!$ndNew) {
                #如果父节点不包含该字符，就新建一个Node,并增加到父节点
                $ndNew = Algorithm::AhoCorasick::Node->new(parent => $nd, char => $c);
                $nd->add_transition($ndNew);
            }   
            #修改父节点
            $nd = $ndNew;
        }   
        #在最后一个节点,增加一个result，为了搜索时可以直接匹配的字符串 
        $nd->add_result($p);
    }   

    #设置第一层的不匹配的转移节点
    my @nodes;
    #循环根节点下的所有节点
    foreach my $nd ($self->{root}->transitions) {
        #第一层的不匹配肯定是根节点
        $nd->failure($self->{root});
        #增加该节点的子节点到数组，为了遍历设置其余节点的不匹配转移节点
        push @nodes, $nd->transitions;
    }   
        
    #循环设置其余的不匹配转移节点
    while (@nodes) {
        my @newNodes;
        foreach my $nd (@nodes) { #循环着一层的节点
            my $r = $nd->parent->failure; #父节点的不匹配跳转节点
            my $c = $nd->char; #该节点的字符

            #如果父节点的失败节点的子节点中包含该字符，跳出while，此时$r是不空的
            #如果不满足上面的情况,就用父节点的失败节点的失败节点，其实根节点的failure是undef的呀
            while ($r && !($r->get_transition($c))) {
                $r = $r->failure;
            }
            if (!$r) {
                $nd->failure($self->{root}); #$r=undef，就代表要用根节点
            } else {
                #寻找父节点的失败节点的包含该字符的子节点
                my $tc = $r->get_transition($c);
                #为该节点设置失败节点
                $nd->failure($tc);

                foreach my $result ($tc->results) {
                    $nd->add_result($result);
                }
            }
            #继续下面一层
            push @newNodes, $nd->transitions;
        }
        @nodes = @newNodes; #再来一遍
    }

    #上面根节点的failure是undef，现在设置一下，为查找时可以统一处理
    $self->{root}->failure($self->{root});
    #初始化状态到根节点
    $self->{state} = $self->{root};
}
```

###查找过程

```
sub feed {
    my ($self, $text, $callback) = @_;

    my $index = 0;
    my $l = length($text);
    while ($index < $l) { #一个一个字符的遍历起来
        my $trans = undef;
        while (1) {
            #从当前状态获取子节点中匹配该字符的节点
            $trans = $self->{state}->get_transition(substr($text, $index, 1));
            #如果状态是根节点或者匹配到了就跳出while
            last if ($self->{state} == $self->{root}) || $trans;
            #不是根节点，也没匹配到，转到失败节点
            $self->{state} = $self->{state}->failure;
        }

        #如果匹配到就把状态转到匹配的这个节点
        if ($trans) {
            $self->{state} = $trans;
        }
        #如果匹配到的状态有返回值，那就调用回调函数
        foreach my $found ($self->{state}->results) {
            my $rv = &$callback($index - length($found) + 1, $found);
            #回调函数不好有返回值的呀，有的话一半就停掉了呀
            if ($rv) {
                return $rv;
            }
        }
        #继续继续
        ++$index;
    }

    return undef;
}
```



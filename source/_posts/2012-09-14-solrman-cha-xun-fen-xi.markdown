---
layout: post
title: "solr慢查询分析"
date: 2012-09-14 13:35
comments: true
sharing: true
footer: true
categories: [Java, Solr]
---

###solr慢请求分析,第一种

```
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=560950&q=*:*&wt=json&rows=50} hits=863948 status=0 QTime=2752
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=756250&q=*:*&wt=json&rows=50} hits=863935 status=0 QTime=2472 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=315450&q=*:*&wt=json&rows=50} hits=863935 status=0 QTime=1183 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=557400&q=*:*&wt=json&rows=50} hits=863935 status=0 QTime=2020 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=315550&q=*:*&wt=json&rows=50} hits=863935 status=0 QTime=1061 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=756400&q=*:*&wt=json&rows=50} hits=863935 status=0 QTime=2252 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=315650&q=*:*&wt=json&rows=50} hits=863938 status=0 QTime=1068 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=557500&q=*:*&wt=json&rows=50} hits=863938 status=0 QTime=2012 
INFO: [] webapp=/solr path=/select/ params={fl=id&sort=quetime+asc&start=557550&q=*:*&wt=json&rows=50} hits=863938 status=0 QTime=2248 
```
###解决方案
+ 请排除以上查询语句【或限制start】


###solr慢请求分析,第二种

```
INFO: [] webapp=/solr path=/select params={mm=2&fl=score,*&sort=score+desc,quetime+desc&start=0&q=贷款+房子+能+不+买&bf=linear(rankscore,29.3246184,0)&qf=title^5+tags^1+bestanswerstr^2+description^1+normalanswerstr^1&wt=json&qt=dismax&fq=-cityid:14&fq=questatus:1+OR+questatus:2+OR+questatus:3+OR+questatus:6&rows=1} hits=317500 status=0 QTime=520 
INFO: [] webapp=/solr path=/select params={mm=2&fl=score,*&sort=score+desc,quetime+desc&start=0&q=集体+户口+好+好+不&qf=title^5+tags^1+bestanswerstr^2+description^1+normalanswerstr^1&wt=json&qt=dismax&fq=-cityid:12&fq=questatus:1+OR+questatus:2+OR+questatus:3+OR+questatus:6&rows=1} hits=287285 status=0 QTime=622 
```
###解决方案
+ dismax的mm=2间接使用了OR，建议用AND提高性能【实验中:我想用docfreq来做判断用AND还是OR】
+ StopWord:通过solr/admin/luke?fl=normalanswerstr&numTerms=100分析term,过滤掉没有意义的term,例如

```
<int name="的">965396</int>
<int name="，">947429</int>
<int name="是">729063</int>
<int name="。">707417</int>
<int name="不">571925</int>
<int name="你">534142</int>
<int name="可以">525645</int>
<int name="就">460975</int>
<int name="有">443743</int>
<int name="了">437630</int>
<int name="要">372936</int>
<int name="！">342369</int>
<int name="好">331646</int>
<int name="在">324636</int>
<int name="一">279765</int>
<int name="个">269726</int>
<int name="能">233510</int>
```

+ 权重相同的放入同一个字段，也可以用CopyField，可以多加个系数

---
layout: post
title: "显示solr省略的日志"
date: 2012-09-27 13:54
comments: true
sharing: true
footer: true
categories: [Java, Solr]
---

###solr日志默认的效果
+ 删除时默认只显示10条记录，其余的使用省略号代替
+ 一般情况下这样是比较好的，但是有时还是希望看到所有的信息

```
webapp=/solr path=/update params={wt=json} 
{delete=[11, 12, 13, 14, 15, 16, 17, 18, 19, 20... (100 deletes)]} 0 127
```

<!--more -->

###测试程序
+ 查询100条记录后删除

```php
<?php
include_once('Apache/Solr/Service.php');
$solr = new Apache_Solr_Service('localhost','8984','/solr');

$rep = $solr->search('*:*',0,100);
$rep = $rep->response->docs;

$ids = array();
foreach($rep as $r){
    var_dump($r->id);
    $ids[] = $r->id;
}
$solr->deleteByMultipleIds($ids);
```

###跟踪solr代码
+ 断点设置在LogUpdateProcessor的processDelete里
+ 小于maxNumToLog才会添加到deletes
+ 多余的部分用省略号表示

```java
  @Override
  public void processDelete( DeleteUpdateCommand cmd ) throws IOException {
    if (logDebug) { log.debug("PRE_UPDATE " + cmd.toString()); }
    if (next != null) next.processDelete(cmd);

    if (cmd.isDeleteById()) {
      if (deletes == null) {
        deletes = new ArrayList<String>();
        toLog.add("delete",deletes);
      }
      if (deletes.size() < maxNumToLog) { //小于maxNumToLog才会添加到deletes
        long version = cmd.getVersion();
        String msg = cmd.getId();
        if (version != 0) msg = msg + " (" + version + ')';
        deletes.add(msg);
      }
    } else {
      if (toLog.size() < maxNumToLog) {
        long version = cmd.getVersion();
        String msg = cmd.query;
        if (version != 0) msg = msg + " (" + version + ')';
        toLog.add("deleteByQuery", msg);
      }
    }
    numDeletes++;

  }

  @Override
  public void finish() throws IOException {
    //略
    if (deletes != null && numDeletes > maxNumToLog) {
      deletes.add("... (" + numDeletes + " deletes)"); //如果大于就要省略号表示
    }
    //略
  }
```

###更改配置，显示所有结果
+ 默认配置中是没有updateRequestProcessorChain，如果没有配置，默认也会加载下面3个处理器
+ RunUpdateProcessorFactory一般必须要有的，作用是更新请求
+ DistributedUpdateProcessorFactory作用是更新shard
+ LogUpdateProcessorFactory作用是记录更新日志
+ maxNumToLog决定日志显示的个数，默认为10
+ 参考[http://wiki.apache.org/solr/UpdateRequestProcessor](http://wiki.apache.org/solr/UpdateRequestProcessor)
+ 参考[http://wiki.apache.org/solr/SolrConfigXml](http://wiki.apache.org/solr/SolrConfigXml)


```
  <updateRequestProcessorChain name="" default="true" >
    <processor class="solr.DistributedUpdateProcessorFactory" />
    <processor class="solr.LogUpdateProcessorFactory" >
      <int name="maxNumToLog">100</int>
    </processor>
    <processor class="solr.RunUpdateProcessorFactory" />
  </updateRequestProcessorChain>
```



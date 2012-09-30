---
layout: post
title: "solr省略日志"
date: 2012-09-27 13:54
comments: true
sharing: true
footer: true
categories: [Java, Solr]
---


http://wiki.apache.org/solr/UpdateRequestProcessor
http://wiki.apache.org/solr/SolrConfigXml
http://lucene.apache.org/solr/api-4_0_0-BETA/org/apache/solr/update/processor/UpdateRequestProcessorChain.html

```
  <updateRequestProcessorChain name="" default="true" >
    <processor class="solr.DistributedUpdateProcessorFactory" />
    <processor class="solr.LogUpdateProcessorFactory" >
      <int name="maxNumToLog">100</int>
    </processor>
    <processor class="solr.RunUpdateProcessorFactory" />
  </updateRequestProcessorChain>
```

---
layout: post
title: "solr查询过程"
date: 2012-08-29 16:25
comments: true
sharing: true
footer: true
categories: [Solr, Java]
---

###solr查询分析
![solr 查询分析](/images/post/solrsearch.png "solr 查询分析")

1. 设置SolrQueryRequest,SolrQueryResponse
1. 在SolrRequestInfo里设置一个ThreadLocal<SolrRequestInfo>,set solrreq,solrresp
1. 通过url取得handler，例如/select就是使用SearchHandler 
1. handler抽象类里的handleRequest
1. handler调用handleRequestBody开始处理请求，建立responseBuilder
1. QueryComponent.prepare，设置returnFields，没有fl的情况下wantsAllFields，设置默认的LuceneQParser分析器，实例化SolrQueryParser
1. QueryComponet.execute，取得关联的文档id，保存到resultCache
1. writeResponse，如果id在DocumentCache里有就直接拿，否则通过reader去获得，再保存到DocumentCache
1. 输出结果到游览器

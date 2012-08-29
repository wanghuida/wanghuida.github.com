---
layout: post
title: "solr启动分析"
date: 2012-08-28 08:34
comments: true
sharing: true
footer: true
categories: [Solr, Java]
---

###solr启动分析
![solr 启动分析](/images/post/solrstart.png "solr 启动分析")

1. CoreContainer通过xpath加载solr.xml
1. 实例化CoreAdminHandler管理CoreContainer，提供reload,status等命令(status可以看到index-size)
1. 初始化CoreDescriptor，设置核心的配置文件、目录、属性文件
1. 读取solrconfig.xml，分析后设置到SolrConfig 
1. 读取schema.xml，设置fieldTypes @FieldTypePluginLoader,fields @SchemaField,过滤器,分词器,主键
1. 构造SolrCore,设置数据目录和各种资源的初始化
1. QuerySenderListener，默认2个事件firstSearcher和newSearcher
1. IndexDeletionPolicy
1. DefaultCodecFactory，4.0的功能，字段的编码器和解码器
1. StandardDirectoryFactory，指向data/index/的Directory，用来生成Reader读取索引
1. QueryResponseWriter，各种返回结果，例如json,xml
1. QParserPlugin，请求分析插件,用以生成QParser，例如BoostQParserPlugin和DisMaxQParserPlugin等
1. ValueSourceParser，初始化内置的函数，比如boost,abs,sin..等等
1. SearchComponent，例如query,facet,mlt,stat,debug,get
1. RequestHandlers，是根据url对应RequestHandler的映射
1. DirectUpdateHandler2，commit数据的handler
1. 最后执行firstSearcher事件预热，注册第一个SolrIndexSearcher

###solr package
+ org.apache.solr.analysis; 过滤器，分词器
+ org.apache.solr.core; solr的核心包,例如CoreContainer,Config,SolrCore,SolrConfig,CachingDirectoryFactory,CoreDescriptor
+ org.apache.solr.servlet; 与servlet有关,例如SolrDispatchFilter就属于该包
+ org.apache.solr.handler; org.apache.solr.handler.component; 请求处理
+ org.apache.solr.handler.admin; 处理用来管理的请求，例如CoreAdminHandler
+ org.apache.solr.search; 与查询有关的类，例如缓存，查询分析，返回字段 SolrIndexSearcher
+ org.apache.solr.schema; 字段类型，schema通知 
+ org.apache.solr.util.plugin; 为插件提供的一些工具，抽象和接口，例如AbstractPluginLoader
+ org.apache.solr.util; 一些工具，例如FileUtils
+ org.apache.solr.update; org.apache.solr.update.processor; 用来更新数据的，例如默认的DirectUpdateHandler2
+ org.apache.solr.response;org.apache.solr.response.transform;用来返回数据(序列化,json,xml等数据)
+ org.apache.solr.request; 请求相关的信息，例如SolrRequestInfo

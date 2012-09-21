---
layout: post
title: "solr的数值存储和范围查询"
date: 2012-09-21 20:49
comments: true
sharing: true
footer: true
categories: [Java, Solr]
---

###问题：solr查询q=date:[1348233109 TO *]，QTime=300毫秒

###要点：lucene内部不存储数值，而用多个字符串代替

####我的疑问：为什么不用数值呀，存储空间少，效率又高? 琢磨了下应该有如下原因吧
+ lucene索引的倒排表都是根据字符串排序的，要存储数值看来得另起一套索引
+ 如果存储数值，查询的排序也不能正常运作了，比如10比2排在前面
+ 查询12,是准确匹配12能还是能匹配123
+ 感觉问题越来越复杂，如果数值也是串，那问题就简单多了,和一般的词没两样

###lucene测试程序
+ 添加id为695和705的两个文档，搜索id区域698到2054 

```java
public static void main(String[] args) throws IOException, ParseException{
    Directory dir = FSDirectory.open(new File("/tmp/tempindex"));
    Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_40);
    IndexWriterConfig iwc = new IndexWriterConfig(Version.LUCENE_40, analyzer);
    iwc.setOpenMode(OpenMode.CREATE);
    IndexWriter writer = new IndexWriter(dir, iwc);
    Document doc = new Document();  
    doc.add(new LongField("id",695,Store.YES));  
    writer.addDocument(doc);  
    doc = new Document();  
    doc.add(new LongField("id",705,Store.YES));  
    writer.addDocument(doc);  
    writer.close();
    
    DirectoryReader reader = DirectoryReader.open(dir);
    IndexSearcher searcher = new IndexSearcher(reader);
    Query q = NumericRangeQuery.newLongRange("id", 698L, 2054L, true, true); 
    ScoreDoc[] hits = searcher.search(q, 200).scoreDocs;  
    for(int i=0; i < hits.length; i++){
      Document result = searcher.doc(hits[i].doc);
      IndexableField id = result.getField("id");
      System.out.println(String.format("doc:%s,id:%s", hits[i].doc, id.numericValue().longValue()));
    }
    reader.close();
    dir.close();
}
```

###调试监测建立索引时，695和705生成的多个字符串
+ 断点设在NumericUtils.java的longToPrefixCoded方法内

```
          695                         705
[20 1 0 0 0 0 0 0 0 5 37]   [20 1 0 0 0 0 0 0 0 5 41] 0x20代表long,低字节bytes高索引位  
[24 8 0 0 0 0 0 0 0 2b]     [24 8 0 0 0 0 0 0 0 2c]   每7位代表一个字符
[28 40 0 0 0 0 0 0 2]       [28 40 0 0 0 0 0 0 2]     每次循环右移precisionStep=4
[2c 4 0 0 0 0 0 0 0]        [2c 4 0 0 0 0 0 0 0]
[30 20 0 0 0 0 0 0]         [30 20 0 0 0 0 0 0]
[34 2 0 0 0 0 0 0]          [34 2 0 0 0 0 0 0]
[38 10 0 0 0 0 0]           [38 10 0 0 0 0 0]
[3c 1 0 0 0 0 0]            [3c 1 0 0 0 0 0]
[40 8 0 0 0 0]              [40 8 0 0 0 0]
[44 40 0 0 0]               [44 40 0 0 0]
[48 4 0 0 0]                [48 4 0 0 0]
[4c 20 0 0]                 [4c 20 0 0]
[50 2 0 0]                  [50 2 0 0]
[54 10 0]                   [54 10 0]
[58 1 0]                    [58 1 0]
[5c 8]                      [5c 8]
```

###698-2057的区域转化
+ 断点设在NumericUtils.java的longToPrefixCoded方法内

```
698: [20 1 0 0 0 0 0 0 0 5 3a]  703: [20 1 0 0 0 0 0 0 0 5 3f] 
704: [24 8 0 0 0 0 0 0 0 2c]    767: [24 8 0 0 0 0 0 0 0 2f]
768: [28 40 0 0 0 0 0 0 3]      2047:[28 40 0 0 0 0 0 0 7]
2048:[20 1 0 0 0 0 0 0 0 10 0]  2054:[20 1 0 0 0 0 0 0 0 10 6]
```
+ 703的生成方法是对698的右边precisionStep位做或运算设置为1
+ 704的生成方法是在703的基础上加1，右移precisionStep位,（其实只要前缀匹配，后面已经无所谓了）
+ 767的生成方法是在704的基础上，右边precisionStep位做或运算设置为1
+ 后面的数字生成方法雷同

###查询索引比较
+ 大于等于698,小于等于703的没有
+ 大于等于704,小于等于767的有，705的第二条记录
+ 大于等于768,小于等于2047的没有
+ 大于等于2048,小于等于2054的没有

###结论：要提高效率的方法
+ 调整precisionStep,默认为4，调大索引小性能差，调小索引大性能理论上的好,精度高，见下图

![rangequery](/images/post/rangequery.gif "rangequery")

+ 如果int够用，可以不用Long用Int
+ 缩小范围，这是管用的方法，（优化业务需求）

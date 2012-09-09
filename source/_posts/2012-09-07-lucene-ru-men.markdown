---
layout: post
title: "lucene 入门"
date: 2012-09-07 22:10
comments: true
sharing: true
footer: true
categories: [Solr, Java]
---

###IndexFiles

+ 在main()方法里分析命令行，然后初始化IndexWriter,打开Directory,初始化StandaryAnalyzer和IndexWriterConfig
+ -index参数设置索引存放的地方，默认存放在index目录下
+ -docs参数指定哪个目录下的文件被索引
+ -update参数如果被设置，将不删除已经存在的索引，否则清楚后重新建立索引
+ IndexWriter使用Directory存储索引信息,这里使用的是FSDirectory实现,其他的子类可以写入RAM,DB
+ Analyzer用来分词(indexed tokens)也称为terms,也包括一些其他的处理，例如统一转换，同义词插入，过滤不需要的词等.现在用的StandaryAnalyzer的分词逻辑是Unicode Text Segmentation algorithm,统一转换成小写，过滤掉不要的词，例如a,an等很少被搜索的词语.不同的语言应该用不同的分析器，lucene已经提供了很多语言的分析器了
+ IndexWriterConfig用来控制IndexWriter,例如设置OpenMode
+ 在IndexWriter初始化后，就来到了indexDocs()方法，这个递归函数爬取-docs下的所有文件来建立Document对象.Document包含3个域分别是文件内容，路径和创建时间.如果-update指定，那么就更新文档，否则删除索引目录后添加文档 


###SearchFiles

+ 下面是一个简易版的文件查询,利用IndexSearcher,StandaryAnalyzer和QueryParser进行查找,QueryParser也需要StandaryAnalyzer对输入的内容进行分析，例如转换小写，过滤词等.QueryParser通过lucene解析语法生成的Query传递给IndexSearcher
+ SearchFiles 用IndexSearcher.search(query,n) 返回得分最高的N条记录.

```java
public class CustomSearch {
  
  public CustomSearch() {}
  
  /**
   * @param args
   */
  public static void main(String[] args) {
    
    String index = "index";
    String field = "contents";
    String queryString = null;
    int limit = 10;
    
    try{
      IndexReader reader = DirectoryReader.open(FSDirectory.open(new File(index)));
      IndexSearcher searcher = new IndexSearcher(reader);
      Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_40);
      QueryParser parser = new QueryParser(Version.LUCENE_40, field, analyzer);
      BufferedReader in = new BufferedReader(new InputStreamReader(System.in,"UTF-8"));
      if(queryString == null){
        System.out.println("Enter query:");
        queryString = in.readLine();
      }
      Query query = parser.parse(queryString);
      printResult(in, searcher, query, limit);
    }catch(IOException e){
      System.out.println("can not open index directory!");
      System.exit(1);
    } catch (ParseException e) {
      System.out.println("can not open index directory!");
      System.exit(2);
    }
  }
  
  public static void printResult(BufferedReader in,IndexSearcher searcher,Query query,int limit) throws IOException{
    
    TopDocs results = searcher.search(query, limit);
    ScoreDoc[] hits = results.scoreDocs;
    int totalHits = results.totalHits;
    int printHits = 0;
    
    while(printHits < totalHits){
      for(int i=0; i < hits.length; i++,printHits++){
        Document doc = searcher.doc(hits[i].doc);
        System.out.println(String.format("score=%s,id=%s,doc=%s,path=%s",hits[i].score, printHits + 1, hits[i].doc, doc.get("path") ));
        if( (i + 1) == limit ){
          System.out.println();
          results = searcher.searchAfter(hits[i], query, limit);
          hits = results.scoreDocs;
        }
      }
    }
  }
  
}
```

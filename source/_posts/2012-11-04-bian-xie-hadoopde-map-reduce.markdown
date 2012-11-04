---
layout: post
title: "编写hadoop的Map-Reduce"
date: 2012-11-04 21:12
comments: true
sharing: true
footer: true
categories: [Hadoop, Java]
---

###创建map-reduce项目

+ 安装hadoop请参考[这篇文章](/blog/2012/11/02/an-zhuang-pei-zhi-hadoop/)
+ 安装hadoop-plugin请参考[这篇文章](/blog/2012/11/03/eclipsepei-zhi-hadoopde-map-reducekai-fa-huan-jing/)

![new-mapred](/images/post/new-mapred.png "new-mapred")


###创建测试数据

```
cd /tmp
mkdir input
echo "hello world baby huida" > t1.txt
echo "world baby baby markdown" > t2.txt

bin/hadoop fs -put /tmp/input input
bin/hadoop fs -ls
```

<!-- more -->

###复制测试代码

+ hadoop源代码里的例子：/src/example/org/apache/hadoop/example/WordCount.java

```java
package org.anjuke.mapred;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {

  public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable>{
    
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
      
    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
    
  }
  
  public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
    if (otherArgs.length != 2) {
      System.err.println("Usage: wordcount <in> <out>");
      System.exit(2);
    }
    Job job = new Job(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
    FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```


###运行实例

+ 修改运行参数，根据自己的路径修改

![wordcount-conf](/images/post/wordcount-conf.png "wordcount-conf")

+ 点击运行后看结果

![mapred-run](/images/post/mapred-run.png "mapred-run")

![wordcount-ret](/images/post/wordcount-ret.png "wordcount-ret")



---
layout: post
title: "WeakHashMap作用和实例"
date: 2012-08-01 21:48
comments: true
sharing: true
footer: true
categories: [Java]
---


###在这里先预想了解一下WeakHashMap，结合后面的例子应该会更好理解
+ 当存放在WeakHashMap里的对象，没有其他引用和使用，那么在GC时会自动释放。

###下面用实例来更好的理解
+ 测试一下保持引用会不会被自动释放

```java
package wanghuida.test;

import java.util.WeakHashMap;
import java.util.Map;

import java.util.List;
import java.util.ArrayList;

public class Entry {

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        List<String[]> templist = new ArrayList<String[]>();  
        //设的多一点，可以让GC真实发挥
        for(int i=0; i < 1000000; i++){
            String[] tempstr = new String[2];
            templist.add(tempstr);
        }
        
        Map<String[], String[]> map = new WeakHashMap<String[], String[]>();
        for(int i=0; i < 100; i++){
            map.put(templist.get(i), new String[2]);
            System.gc();
            System.out.println(map.size());
        }

    }

}
```

<!-- more -->
+ 输出1，2，3，4。。。递增，OK没有问题，有一个引用，就不会释放
+ 再测试一下删除引用后的总数

```java
package wanghuida.test;

import java.util.WeakHashMap;
import java.util.Map;

import java.util.List;
import java.util.ArrayList;

public class Entry {

    public static void main(String[] args) {
        List<String[]> templist = new ArrayList<String[]>();  
        //设的多一点，可以让GC真实发挥
        for(int i=0; i < 1000; i++){
            String[] tempstr = new String[2];
            templist.add(tempstr);
        }
        
        Map<String[], String[]> map = new WeakHashMap<String[], String[]>();
        for(int i=0; i < 100; i++){
            map.put(templist.get(i), new String[2]);
            templist.set(i, null); //删除掉引用 
            System.gc();
            System.out.println(map.size());
        }

    }

}
```

+ 输出0，1，0，1，1，1。。。保持下去，OK也没问题，删除引用后就会释放
+ 再测试一下有引用但没有使用的情况

```java
package wanghuida.test;

import java.util.WeakHashMap;
import java.util.Map;

import java.util.List;
import java.util.ArrayList;

public class Entry {

    public static void main(String[] args) {
        List<String[]> templist = new ArrayList<String[]>();  
        //新增一个引用
        List<String[]> list = new ArrayList<String[]>();  
        //设的多一点，可以让GC真实发挥
        for(int i=0; i < 1000000; i++){
            String[] tempstr = new String[2];
            templist.add(tempstr);
            list.add(tempstr);
        }
        
        Map<String[], String[]> map = new WeakHashMap<String[], String[]>();
        for(int i=0; i < 100; i++){
            map.put(templist.get(i), new String[2]);
            templist.set(i, null); //删除掉引用 
            System.gc();
            System.out.println(map.size());
        }

    }

}
```

+ 输出0，1，0，1，1，1。。。保持下去，OK也没问题，有引用但不使用也就会释放
+ 最后对比一下有使用的情况

```java
package wanghuida.test;

import java.util.WeakHashMap;
import java.util.Map;

import java.util.List;
import java.util.ArrayList;

public class Entry {

    public static void main(String[] args) {
        List<String[]> templist = new ArrayList<String[]>();  
        //新增一个引用
        List<String[]> list = new ArrayList<String[]>();  
        //设的多一点，可以让GC真实发挥
        for(int i=0; i < 1000000; i++){
            String[] tempstr = new String[2];
            templist.add(tempstr);
            list.add(tempstr);
        }
        
        Map<String[], String[]> map = new WeakHashMap<String[], String[]>();
        for(int i=0; i < 100; i++){
            map.put(templist.get(i), new String[2]);
            templist.set(i, null); //删除掉引用 
            System.gc();
            System.out.println(map.size());
        }
        
        System.out.println(list.size());

    }

}
```

+ 输出1，2，3，4。。。递增，OK了，希望对大家会有帮助

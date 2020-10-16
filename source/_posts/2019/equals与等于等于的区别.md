---
title: equals与==的区别
permalink: equals与等于等于的区别
date: 2019-11-26 22:14:52
toc: true
tags:
 - Java
categories:
 - Java基础
---

### equals与==的区别



一、基本数据类型 用(==) 进行比较的时候，比较的实际值 

二、包装数据类型 用(==)进行比较的时候，比较的是在内存中存放的地址

三、包装类型中的equals方法，（String，Integer，Date）等重写了equals方法，比较的是地址和内容，地址相同返回true，地址不同但值相同返回true，其他返回false。没有重写equals方法的，比较的还是内存地址。

四、StringBuffer 和 StringBuilder 比较特殊， == 和 equals都是比较的地址。





-------



推荐文章

https://blog.csdn.net/qq_36522306/article/details/80550210

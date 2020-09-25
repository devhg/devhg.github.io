---
title: 什么是IOC和AOP
date: 2019-09-26 10:33:06
toc: true
tags:
 - Java
categories:
 - Java后端
---





### IOC

IOC（Inversion Of Controll，控制反转）是一种设计思想，将原本在程序中手动创建对象的控制权，交由给Spring框架来管理。IOC容器是Spring用来实现IOC的载体，IOC容器实际上就是一个Map(key, value)，Map中存放的是各种对象。

这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂，当需要创建一个对象，只需要配置好配置文件/注解即可，不用考虑对象是如何被创建出来的，大大增加了项目的可维护性且降低了开发难度。

### AOP

AOP（Aspect-Oriented Programming，面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量，提高了系统的扩展性。

Spring AOP是基于动态代理的，如果要代理的对象实现了某个接口，那么Spring AOP就会使用JDK动态代理去创建代理对象；而对于没有实现接口的对象，就无法使用JDK动态代理，转而使用CGlib动态代理生成一个被代理对象的子类来作为代理。

### Spring AOP / AspectJ AOP 的区别？

Spring AOP属于运行时增强，而AspectJ是编译时增强。

Spring AOP基于代理（Proxying），而AspectJ基于字节码操作（Bytecode Manipulation）。

AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单。如果切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比SpringAOP快很多。
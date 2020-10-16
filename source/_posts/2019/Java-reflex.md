---
title: Java反射机制深入学习
permalink: Java-reflex
date: 2019-09-30 19:29:07
toc: true
tags:
 - Java
categories:
 - Java基础
---

**反射是框架设计的灵魂（使用的前提条件：必须先得到代表的字节码的Class，Class类用于表示.class文件（字节码）**

<!--more-->


### 一、反射的概述

Java反射机制是在运行状态中，对于任意一个类(class文件)，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

要想解剖一个类,必须先要获取到该类的字节码文件对象。而解剖使用的就是**Class类**中的方法。所以先要获取到每一个字节码文件对应的Class类型的对象。

反射就是把java类中的各种成分映射成一个个的Java对象。例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把一个个组成部分映射成一个个对象。

（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）。

**以上的总结就是什么是反射**

<br>

如图是类的正常加载过程：反射的原理在与class对象。

熟悉一下加载的时候：Class对象的由来是将class文件读入内存，并为之创建一个Class对象。



### 二、获取字节码对象的三种方式。

```java
Class class1 = Class.forName("cn.coder.Person");　// 常用的
//通过Class类中的静态方法forName，直接获取到一个类的字节码文件对象，此时该类还是源文件阶段，并没有变为字节码文件。

Class class2 = Person.class;
//当类被加载成.class文件时，此时Person类变成了.class，在获取该字节码文件对象，也就是获取自己， 该类处于字节码阶段。

Person person = new Preson();
Class class3 = person.getClass();　

//通过类的实例获取该类的字节码文件对象，该类处于创建对象阶段　
```

有了字节码文件对象才能获得类中所有的信息，我们在使用反射获取信息时，也要考虑使用上面哪种方式获取字节码对象合理，视不同情况而定。



### 三、优雅的利用字节码对象

下面通过一个实例介绍Class类的功能

#### 3.1通过字节码对象获取构造器实例对象

```java
// 第一种
Class class1 = Class.forName("cn.coder.Person");
Person p1 = (Person)class1.newInstance(); // 通过无参构造器 创建Person实例
/*
局限：
该类无参的构造方法来是使用该Class类的newInstance()方法来创建对象的, 如果一个类没有无参的构造函数, 就不能这样创建了。
*/

// 第二种
Class class2 = Class.forName("cn.coder.Person");
Construnctor con = class2.getConstructor(int.class, String.class);
Construnctor con0 = class2.getConstructor(new Class[] {int.class, String.class}); //效果同上
Person p2 = (Person)con.newInstance(18,"臭弟弟");
Person p0 = (Person)con0.newInstance(new Object[]{18, "臭弟弟"}); // 效果同上

// 第三种   其实还可以  获取所有构造器
Constructor[] con = class2.getConstructors();
for(int i=0; i<con.length; i++){
  //获取每个构造函数中的参数类型字节码对象
  Class[] parameterTypes = con[i].getParameterTypes();

  for(int j=0; j<paraparameterTypes.length; j++){
    //获取每个构造函数中的参数类型
    System.out.print(paraparameterTypes[j].getName() + ",");
  }

}
```

<br>

#### 3.2获取成员变量并使用---Filed对象

```java
Class personClass = Person.class;

// 获取类变量
Field[] fields = personClass.getFields(); // 只能获取public修饰的变量
Field[] fields1 =personClass.getDeclaredFields(); // 获取所有变量

System.out.println("---------------");
Field a = personClass.getField("a");  // 获取名为a的变量

Person p = new Person();
Object value = a.get(p); // 获取p对象中a的值
a.set(p, "111"); // 设置p对象中a的值

System.out.println("---------------");
Field d = personClass.getDeclaredField("d");
// 假设d是private的  则不能get  set
// 那么要开启set  get权限  如下语句开启
d.setAccessible(true);

```

<br>

#### 3.3获得方法并使用 Method

**和上面类似 Fileld相似的**

可以通过Class.getMethod(String, Class...) 和 Class.getDeclaredMethod(String, Class...)方法可以获取类中的指定方法或者所有方法，获取后可进行遍历。如果为私有方法，则需要打开一个权限。setAccessible(true); 用invoke(Object, Object...)可以调用该方法，跟上面同理，也能一次性获得所有的方法：

```java
Class p2 = Class.forName("com.demo.Reflect.Person"); // 创建字节码对象
Person person = (Person) p2.newInstance(); // 通过字节码实例一个对象

Method[] method2 = p2.getDeclaredMethods(); // 获取所有method数组
for (Method method : method2) {
  method.setAccessible(true); // 开启权限
  System.out.println(method.getName());
  // 获取对象的参数 并遍历打印
  Class<?>[] parameterTypes = method.getParameterTypes();
  for (int i = 0; i < parameterTypes.length; i++) {
    System.out.print(parameterTypes[i].getName() + ",");
  }
	// 通过invoke(实例对象, 参数....)
  method.invoke(person, 3);
}


```



#### 3.4获得该类的所有接口

Class[] getInterfaces()：确定此字节码对象所表示的类或接口实现的接口

返回值：**接口的字节码文件对象的数组**



#### 3.5获取指定资源的输入流

InputStream getResourceAsStream(String name)　　

return：一个 InputStream 对象；如果找不到带有该名称的资源，则返回 null

参数：所需资源的名称，如果以"/"开始，则绝对资源名为"/"后面的一部分。



#### 四、一个整体的实例

```java
package me.zhangsanfeng;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Reflection {
  public Object copy(Object object) throws Exception{
    //通过传入的Object得到该实例对应的类的Class对象
    Class<?> classType = object.getClass();

    //通过Class得到该类的Constructor对象
    Constructor<?> cons = classType.getConstructor(new Class[]{});

    //通过反射机制使用构造方法得到一个对象，用来接受复制的内容
    Object retObject = cons.newInstance(new Object[]{});

    //取到实例对应的类的所有成员变量
    Field[] fields = classType.getDeclaredFields();

    for(Field field:fields){
      String nameOfField = field.getName();
      /*组装成员变量的get、set方法*/
      String fristWord = nameOfField.substring(0, 1).toUpperCase();
      String getMethodName = "get"+fristWord+nameOfField.substring(1);
      String setMethodName = "set"+fristWord+nameOfField.substring(1);
      //通过反射创建类中方法的对象
      Method getMethod = classType.getMethod(getMethodName, new Class[]{});
      Method setMethod = classType.getMethod(setMethodName, new Class[]{field.getType()});
      /*使用get方法从传入对象取值，使用set方法将取出的值赋给等待复制的retObject*/
      Object value = getMethod.invoke(object, new Object[]{});
      setMethod.invoke(retObject, new Object[]{value});
    }
    return retObject;
  }

  public static void main(String[] args) throws Exception {
    //使用反射创建一个person实例
    Class<?> classType = Person.class;
    Constructor cons = classType.getConstructor(new Class[]{String.class,int.class,double.class});
    //使用带参数的构造方法创建对象
    Object perOfLee = cons.newInstance(new Object[]{"Lee",25,8000});

    //使用反射调用copy方法
    Class <?> classOfReflect = Class.forName("me.zhangsanfeng.Reflection");
    Object reflect = classOfReflect.newInstance();
    Method methodOfCopy = classOfReflect.getMethod("copy", new Class[]{Object.class});
    Object finalObject = methodOfCopy.invoke(reflect, perOfLee);
    Person copyPerson = (Person)finalObject;

    //print
    System.out.println(copyPerson.getName()+", "+copyPerson.getAge()+", "+copyPerson.getSalary());
  }
}

//定义一个“人”类
class Person{
  private String name;
  private int age;
  private double salary;

  public Person() {

  }

  public Person(String name, int age, double salary){
    this.name = name;
    this.age = age;
    this.salary = salary;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  public double getSalary() {
    return salary;
  }

  public void setSalary(double salary) {
    this.salary = salary;
  }

}
```


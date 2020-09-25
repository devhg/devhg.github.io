---
title: Java注解学习
date: 2019-10-07 21:28:44
toc: true
tags:
 - Java
categories:
 - Java基础
---

注释：用文字描述程序，给程序员看的

注解：给机器看的

<!--more-->

## 1.注解基础知识点

定义：注解（Annotation），也叫元数据。一种代码级别的说明。它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

### 1.1作用分类：

①编写文档：通过代码里标识的注解生成文档【生成文档doc文档】

②代码分析：通过代码里标识的元数据对代码进行分析【使用反射】

③编译检查：通过代码里标识的元数据让编译器能够实现基本的编译检查，编译检查是否是复写父类的方法【Override】

给某个类、方法..添加了一个注解，这个环节仅仅是做了一个标记，对代码本身并不会造成任何影响，需要后续环节的配合，需要其他方法对该注解赋予业务逻辑处理。就如同我们在微信上发了一个共享定位，此时并没有什么用，只有当后面其他人都进入了这个共享定位，大家之间的距离才能明确，才知道该怎么聚在一起。

### 1.2注解分为三类：

#### 1.2.1编译器使用到的注解

如@Override，@SuppressWarnings都是编译器使用到的注解，作用是告诉编译器一些事情，而不会进入编译后的.class文件。

@Override：告诉编译器检查一下是否重写了父类的方法；

@SuppressWarnings：告诉编译器忽略该段代码产生的警告；

对于开发人员来说，都是直接使用，无需进行其他操作

#### 1.2.2.class文件使用到注解

需要通过工具对.class字节码文件进行修改的一些注解，某些工具会在类加载的时候，动态修改用某注解标注的.class文件，从而实现一些特殊的功能，一次性处理完成后，并不会存在于内存中，都是非常底层的工具库、框架会使用，对于开发人员来说，一般不会涉及到。

#### 1.2.3运行期读取的注解

一直存在于JVM中，在运行期间可以读取的注解，也是最常用的注解，如Spring的@Controller，@Service，@Repository，@AutoWired，Mybatis的@Mapper，Junit的@Test等，这类注解很多都是工具框架自定义在运行期间发挥特殊作用的注解，一般开发人员也可以自定义这类注解。

## 2.自定义注解：

### 2.1格式：

```java
public @interface 注解名称 {	
  //属性列表：
}  
```



### 2.2本质：

 注解的本质就是一个接口，该接口默认继承java.lang.annotation.Annotation 

public interface myAnnotation extends java.lang.annotation.Annotation {
}



### 2.3属性

接口中可以自定的成员方法   ===  抽象方法

返回类型要求： 不能是包装类型

1. 8种基本数据类型
2. String
3. 枚举
4. 注解
5. 以上类型的数组

要求：

1. 定义了属性，在使用时需要给属性赋值 （可以设置默认值） String name() default "";

2. 特殊的属性  只有一个属性需要赋值，且属性的名字的是value   书写注解的时候可以省略vlaue=  

   > @myAnnotation(name = "辉")  ==>  @myAnnotation("辉")

3. 数组赋值的时候，使用{}包裹。如果数组中只有一个值，则{}省略

注解样例：

```java
package com.demo.Annotation;

public @interface myAnnotation {
    // 下面代表一个个的属性  default代表默认
    String value() default "";
    int age() default 0;
    Person p();
    myAnno2 ANNO_2();
    int[] Arr();
}
```

使用注解：

```java
package com.demo.Annotation;

@myAnnotation(value = "xx", age = 11, p = Person.p1, ANNO_2 = @myAnno2, Arr = {1, 2})
public class woker {
}

```



### 2.4元注解

定义： 用于描述注解的注解  元注解也可以描述元注解

@Target： 描述注解能够作用的位置

```java
@Target(ElementType.TYPE) // 只能作用于类
@Target(ElementType.METHOD) // 只能作用于方法
@Target(ElementType.FIELD) // 只能作用于字段
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD}) // 作用于多个
//.....other
```

@Retention：描述注解被保留的阶段

```java
@Retention(RetentionPolicy.RUNTIME)
@Retention(RetentionPolicy.CLASS)
@Retention(RetentionPolicy.SOURCE)

//-------------------------------------------------------//
public enum RetentionPolicy {
    /**
     * 仅存在于源代码中，编译阶段会被丢弃，不会包含于class字节码文件中.
     */
    SOURCE,

    /**
     * 【默认策略】，在class字节码文件中存在，在类加载的时被丢弃，运行时无法获取到
     */
    CLASS,

    /**
     * 始终不会丢弃，可以使用反射获得该注解的信息。自定义的注解最常用的使用方式。
     */
    RUNTIME
}
```



@Documented：描述注解是否会被抽取到文档中

表示是否将此注解的相关信息添加到javadoc文档中

@Inherited：描述注解是否被子类继承

定义该注解和子类的关系，使用此注解声明出来的自定义注解，在使用在类上面时，子类会自动继承此注解，否则，子类不会继承此注解。注意，使用@Inherited声明出来的注解，只有在类上使用时才会有效，对方法，属性等其他无效。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface Person {
    String value() default "man";
}
```

```java
@Person
public class Parent {
}
//子类也拥有@Person注解
class Son extends Parent {

}
```

### 2.5定义注解小结

用@interface定义注解

可以添加多个参数，核心参数按约定用value，为每个参数可以设置默认值，参数类型包括基本类型、String和枚举

可以使用元注解来修饰注解，元注解包括多个，必须设置`@Target`和`@Retention`，`@Retention`一般设置为`RUNTIME`。



## 3.Annotation解析处理

### 3.1解析处理

含注解的类

```java
package com.demo.Annotation;

public class User {
    @Colum("辉")
    private String name;

    @Colum("18")
    private String age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }
}

```

获取注解

```java
@Test
public void test02() throws ClassNotFoundException {
  List<String> colums = new ArrayList<>();
  // 获取目标类的字节码
  Class<?> clazz = Class.forName("com.demo.Annotation.User");
  // 获取该类中所有字段
  Field[] fields = clazz.getDeclaredFields();

  for (Field field : fields) {
    // 获取类中每一个字段的Colum注解
    Colum col = field.getAnnotation(Colum.class);
    // 或者可以先判断有无该注解
    field.isAnnotationPresent(Colum.class);
    // 添加进集合
    colums.add(col.value());
  }
  // 打印集合
  colums.forEach((name) -> System.out.println(name));
}
```

比如我们有一些常见的应用场景，需要把网站上的列表导出成excel表格，我们通过注解的方式把列名配置好，再通过反射读取实体需要导出（是否需要导出，也可通过注解配置）的每个字段的值，从而实现 excel导出的组件。



### 3.2注解底层原理

注解定义后也是一种class，所有的注解都继承自`java.lang.annotation.Annotation`，因此，读取注解，需要使用反射API。

```java
package com.demo.Annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Colum {
    String value() default "";
    String name() default "";
}
```



**其实只要在类上、方法上、字段上声明注解后**

**就会根据注解上写的相关的value值生成一个.class**

上面注解生成一个 注解接口的实现类

```java
//....
public class ColumImpl implements Colum {
  	public String getValue() {
      	return "声明注解时填写的内容";
    }
  	public String getName() {
      	return "声明注解时填写的内容";
    }
}

// 然后 获取注解接口实现类实例对象 col
Colum col = field.getAnnotation(Colum.class);
// 然后就可以通过对象储存的值 和 强大的反射 进行一系列操作了

```

## 4.总结

本文只是抛砖引玉地讲解了注解的基本概念，注解的作用，几种元注解的功用以及使用方法，并通过一个简单的例子讲解了一下注解的处理，并不全面，文中通过Field讲解了注解的基本Api，但注解还可以修饰类、构造器、方法等，也有相对应的注解处理方法，大家可自行查一下API手册相关内容，大同小异，有不对之处，请批评指正，望共同进步，谢谢！
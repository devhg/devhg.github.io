---
title: Mybatis查询parameterType多参数问题
permalink: Mybatis查询parameterType多参数问题
date: 2019-08-15 21:32:45
tags:
 - Java
categories:
 - Java后端
---

## Mybatis查询parameterType多参数问题





### 一、单个参数：

```java

List<EntInfo> findByUsername(String username);

<select id="findByUsername" resultMap="EntInfoResultMap" parameterType="java.lang.String" >
.......
</select>
```

其中方法名和ID一致，#{} 中的参数名与方法中的参数名一致， 这里采用的是@Param这个参数，实际上@Param这个最后会被Mabatis封装为map类型的。

select 后的字段列表要和bean中的属性名一致， 如果不一致的可以用 as 来补充。





### 二、多参数：

方案1

```java
public List<XXX> getXXX(String xxId, String xxCode);  

<select id="getXXX" resultType="XXX"> // 不需要写parameterType参数

　　select t.* from tableName where id = #{0} and name = #{1}  

</select>  

// 由于是多参数那么就不能使用parameterType， 改用#｛index｝是第几个就用第几个的索引，索引从0开始
```

方案2（推荐）基于注解

```java
public List<XXXBean> getXXXBeanList(@Param("id")String id, @Param("code")String code);  

<select id="getXXXBeanList" resultType="XXBean">

　　select t.* from tableName where id = #{id} and name = #{code}  

</select>  

```

由于是多参数那么就不能使用parameterType， 这里用@Param来指定哪一个。



### 三、map封装多参数：

通过传入map对象查询 并返回user对象的list集合  

直接通过map里面的key直接访问  #{key}



```java
public List<User> findUsersByMap(Map<String, Object> map);

<!-- 3. 通过传入map对象查询 并返回user对象的list集合  map里面的属性直接访问 -->
<select id="findUsersByMap" parameterType="hashmap" resultType="user">
    select * from users where sex=#{sex} and username =#{name}
</select>
```



### 四、List封装多个参数：

```java
public List<XXX> getXXX(List<String> list);  

<select id="getXXXBeanList" resultType="XXBean">
　　select 字段... from XXX where id in
　　<foreach item="item" index="index" collection="list" open="(" separator="," close=")">  
　　　　#{item}  
　　</foreach>  
</select>  

// foreach 最后的效果是select 字段... from XXX where id in ('1','2','3','4') 
```




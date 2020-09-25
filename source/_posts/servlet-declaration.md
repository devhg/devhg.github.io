---
title: Servlet生命周期
date: 2019-09-19 22:08:42
Toc: true
tags:
 - Java
categories:
 - Java后端
---



**生命周期：** **Web容器加载Servlet并将其实例化后，Servlet生命周期开始**，容器运行其**init()方法**进行Servlet的初始化；请求到达时调用Servlet的**service()方法**，service()方法会根据需要调用与请求对应的**doGet或doPost**等方法；当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用Servlet的**destroy()方法**。**init方法和destroy方法只会执行一次，service方法客户端每次请求Servlet都会执行**。Servlet中有时会用到一些需要初始化与销毁的资源，因此可以把初始化资源的代码放入init方法中，销毁资源的代码放入destroy方法中，这样就不需要每次处理客户端的请求都要初始化与销毁资源。



综上：

Servlet生命周期分为三个阶段：

1. 初始化阶段  调用init()方法,  只执行一次

```
--默认情况下，第一次被访问时，Servlet被创建，然后执行init方法；

--可以配置执行Servlet的创建时机；

--可以配置执行Servlet的创建时机；
```

　	

2. 响应客户请求阶段  调用service()方法 处理doGet和doPost方法，执行多次

3. 终止阶段　当Servlet服务器正常关闭时，执行destroy方法，只执行一次

<br>

代码：

 ```java
//servlet生命周期，的三个方法，
//1.被创建，执行且只执行一次init方法，
//2.提供服务，执行service方法，执行多次 
//3.被销毁，当Servlet服务器正常关闭时，执行destroy方法，只执行一次。

@Override
public void init() throws ServletException {
  // TODO Auto-generated method stub
  super.init();
}

@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  // TODO Auto-generated method stub
  super.service(req, resp);
}

@Override
public void destroy() {
  // TODO Auto-generated method stub
  super.destroy();
}
 ```


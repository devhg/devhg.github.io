---
title: 表单onsubmit事件无效&表单onsubmit后使用ajax无效解决
date: 2019-08-17 20:02:47
toc: true
tags:
  - Web移动端
categories:
 - HTML5+APP
---

这段时间在写一个h5+app，用于一个组队比赛项目的app。由于自身也没有多少前端开发的经验，也就是闷着头一直乱写，在开发中也遇到了各种难题。

今天又解决一发问题，(此处需要掌声)， 成就感！



### 关于登录表单submit的问题



这个过程需要用到的技术主要就是ajax技术和js的onsubmit技术。

过程如下：

> 1.用户输入  用户名和密码
>
> 2.当用户点击提交按钮时，利用ajax请求后端接口 进行用户名密码验证。



但是这时候出现一个问题，就是当 用户点击输入法的  **发送**   **提交** 按钮的时候，我们怎么验证

这时候就需要原生js的 onsubmit 方法了

> 3.1  验证不通过，onsubmit返回false，表单无法提交，页面提示用户密码有错。
>
> 3.2  验证通过，onsubmit返回true，表单提交，服务器返回用户内部视图，登陆成功。





废话不说上代码

```javascript
// 点击手机键盘  提交按钮
document.getElementById("login").onsubmit = function() {
  subData();
};
// 点击登录按钮
document.getElementById("login-btn").addEventListener('tap', function() {
  subData();
})


function subData() {
  // 获取表单数据
  var data = getFormData("login");
  
  mui.ajax('http://192.168.1.142:8080/user/login.do', {
    data: JSON.stringify({
      'username': data.username,
      'password': data.password
    }),
    dataType: 'json', //服务器返回json格式数据
    type: 'post', //HTTP请求类型
    timeout: 10000, //超时时间设置为10秒；
    
    //  async: false, // 同步方式是为了解决   手机键盘提交按钮 提交表单验证bug的
    
    contentType: 'application/json;charset=utf-8', // 少了会报错
    success: function(data) {
      console.log(JSON.stringify(data));
      if (data.status == 200) {
        mui.toast("登录成功")
      } else if (data.status == -200) {
        mui.toast("登录失败，请重新登录")
      }
    },
    error: function(xhr, type, errorThrown) {
      console.log("error");
    }
  });
};
```



 如果用上面的代码，点击手机键盘 提交按钮 是无法完成验证操作的、



### 原因

上面代码用的是  异步的方式

```html
<script type="text/javascript">
  function checkpwd(){
    //1............
    $.ajax({
      //2........
    });
    //3.........
  }
</script>
```

<br>

* 如果是同步方式：当1执行完毕后，接着执行ajax，线程会处于等待状态，等2执行完毕之后，接着执行3.

* 如果是异步方式：当1执行完毕之后，接着执行ajax，但是ajax不会阻塞主线程，ajax执行的同时会执行3.

<br>

下面展示错误的ajax验证方式：

```javascript
// 点击手机键盘  提交按钮
document.getElementById("login").onsubmit = function() {
  // 1........
  
  mui.ajax('http://192.168.1.142:8080/user/login.do', {
    // 2.......
    data: JSON.stringify({
      'username': data.username,
      'password': data.password
    }),
    dataType: 'json', //服务器返回json格式数据
    type: 'post', //HTTP请求类型  
    contentType: 'application/json;charset=utf-8', // 少了会报错
    success: function(data) {
      console.log("success");
    },
    error: function(xhr, type, errorThrown) {
      console.log("error");
    }
  });
  
  // 3........
};
```

<br>

**上面的代码， 先执行 1   后  执行 2  。但是2 (ajax) 不会阻塞主线程，2 (ajax)  执行的同时会执行3.  这时候3 没等验证成功，立马返回false**

因此，当使用异步方式进行验证的时候，会出现无论如何，onsubmit（onclick）都不会起作用，这会让程序员感觉自己的代码有问题，其实代码没问题，是逻辑的问题。要解决这个问题，我们就必须用ajax的同步方式。

<br>

### 解决方案

改为同步方式 ajax  中添加

```javascript
async : false,
```



<br>

<br>



### 参考文章

- [文章1](https://my.oschina.net/qkmc/blog/872778)
- []()
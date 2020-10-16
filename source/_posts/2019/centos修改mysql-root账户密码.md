---

title: centos修改mysql root账户密码
permalink: centos修改mysql-root账户密码
date: 2019-09-05 20:04:29
Toc: true
tags:
 - Mysql
categories:
 - Mysql
---



### 第一步修改my.cnf文件

1.  vim /etc/my.cnf

2. 在[mysqld]中添加    skip-grant-tables

例如：

```properties
[mysqld]
skip-grant-tables # 添加这行
# skip-grant-tables=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
```

重启mysql

```bash
service mysql restart
```

<br>

### 第二步用户无密码登录

```mysql
mysql -uroot -p (直接点击回车，密码为空)
```

<br>

### 第三步选择数据库修改root密码

```mysql
use mysql;
update mysql.user set authentication_string=password('新密码') where User='用户';
```

<br>

### 第四步刷新并退出

```mysql
flush privileges;
quit;
```



<br>

### 第五步编辑my.cnf并重启mysql

<br>

```bash
vim /etc/my.cnf
# 删除 skip-grant-tables  保存退出
service mysql restart  # 重启mysql
```







<br>

<br>

### 参考文章

- [文章1](http://www.jb51.net/article/100211.htm)
- [文章2](https://www.cnblogs.com/jekaysnow/p/8849533.html)

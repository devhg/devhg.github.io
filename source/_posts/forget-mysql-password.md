---
title: Mysql忘记密码解决方案
date: 2019-09-04 21:02:38
Toc: true
tags:
 - Mysql
categories:
 - Mysql
---



Mysql 忘记了密码  解决方案：

<br>

1. Cmd  --  > 停止mysql服务  （以管理员的身份） net stop mysql;
2. 使用无验证方式启动mysql   mysqld --skip-grant-tables
3. 打开新的cmd窗口，直接输入mysql  回车 登录成功
4. 依次执行 use mysql;  set  password  for  "root"@"localhost" = password("新密码");
5. 手动执行mysqld的进程  
6. 启动mysql服务  net  start  mysql;
7. 使用新密码登录


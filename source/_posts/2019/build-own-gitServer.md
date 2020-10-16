---
title: 搭建自己的Git服务器并部署Hexo站点
permalink: build-own-gitServer
date: 2019-12-15 18:44:57
toc: true
tags:
  - Git
  - Hexo
categories:
  - Git

---

## 前言

以前自己的博客站点是托管在Github的，使用过Netlify，coding，但速度都不是很理想。也曾使用过Gitee(速度虽然不从，但绑定域名要付费99， 放弃 )。最近买了阿里云的学生机来玩，国内速度还不错，1核2G5Mbps。所以今天计划把自己原来托管在github的网站迁移到阿里云服务器。迁移过程还算顺利。

我迁移大体的思路是，继续用hexo部署。搭建自己的git服务器，然后用hexo部署到自己的git服务器。然后通过nginx挂载git仓库，实现全静态访问。速度可想而知。

开始操作

## 搭建Git服务器

### 安装git

```bash
$ yum install -y git
```

### 创建仓库

先创建一个`git`用户，用来运行`git`服务：

```bash
$ adduser git # 添加git用户 用于ssh
$ passwd git # 设置刚添加的用户密码 用于以后的push 和 clone
```

- 在 `home/git` 的目录下，创建一个名为`hexoBlog`的裸仓库（bare repo）。

```bash
$ cd /home/git/
$ git init --bare hexoBlog.git # git会创建一个裸仓库
```

上面已经创建的裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以`.git`结尾。然后，把owner改为`git`用户

```bash
$ sudo chown -R git:git /home/git/hexoBlog.git
```

禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑`/etc/passwd`文件完成。找到类似下面的一行：

```bash
git:x:1001:1001:,,,:/home/git:/bin/bash
```

改为：

```bash
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

克隆远程仓库：

现在，可以通过`git clone`命令克隆远程仓库了，在自己的电脑上运行：

```bash
$ git clone git@server:/home/git/hexoBlog.git # server 为ip
Cloning into 'hexoBlog'...
warning: You appear to have cloned an empty repository.
```

出现上面信息，基本说明搭建完毕，剩下的推送就简单了。



### 创建Hooks钩子

创建hooks钩子的目的是把，git仓库里的代码拉到一个路径下，便于查看修改和以后的nginx挂载

在hooks下创建**post-receive**脚本，它将在仓库接收到push时执行。

```bash
$ vim /home/git/hexoBlog.git/hooks/post-receive

# 添加如下内容

#!/bin/bash 
git --work-tree=/home/hexoBlog --git-dir=/home/git/hexoBlog.git checkout -f
echo '拉取完毕'

# 修改完毕，给足权限 巨坑
$ chmod +x /home/git/hexoBlog.git/hooks/post-receive

# 创建 /home/hexoBlog 存放源码
$ mkdir /home/hexoBlog
```

写到这里，用户组对/home/hexoBlog路径只有读的权限，没有写的权限。上边的配置都没有什么问题，就这个权限折腾了一天，用户组默认的权限是没有写权限的，配置好不能上传代码，问题就在用户组的权限。
修改目录及其子文件的权限

```bash
$ chmod -R 777 /home/hexoBlog # 让所有用户有操作权限
```

之后push之后就可以在 /home/hexoBlog看到push的文件了

<hr>

## 部署hexo网站

###  云服务器端配置 Nginx

安装 启动 测试Nginx

```bash
$ yum install -y nginx
# 启动 Nginx
$ systemctl start nginx
# 测试 Nginx 服务器
$ wget http://127.0.0.1
```

能够正常获取以下欢迎页面说明Nginx安装成功。

```javascript
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 43704 (43K) [text/html]
Saving to: ‘index.html’

100%[=======================================>] 43,704      --.-K/s   in 0s

2019-12-15 23:04:09 (487 MB/s) - ‘index.html’ saved [43704/43704]
```

查看 Nginx 的默认配置的安装位置

 nginx -t 

修改Nginx的默认配置，其中 vim 后边就是刚才查到的安装位置（每个人可能都不一样）

```bash
$ vim /etc/nginx/nginx.conf 

# 找到location / {}
# root为静态文件存放地址 也就是仓库
location / {
  root /home/hexoBlog;
  index index.html;
}
```

重启 Nginx 服务

```bash
$ systemctl restart nginx
```



至此，服务器端配置就结束了。接下来，就剩下本地 hexo 的配置更改了。



### 修改 hexo 站点配置文件 git 相关设置

打开你本地的 hexo 博客所在文件，打开站点配置文件（不是主题配置文件），做以下修改。

```yaml
deploy:
    type: git
    repo: git@你的云服务器的IP地址:/home/git/hexoBlog
    branch: master
```

在 hexo 目录下执行部署，试试看。

```bash
$ cd 你的hexo目录
$ hexo clean
$ hexo g -d 
```

打开你的公网 IP，看是不是已经部署成功了。



#### 参考文章

* [Hexo 博客部署到腾讯云教程](https://cloud.tencent.com/developer/article/1140005)
* [带你跳过各种坑，一次性把 Hexo 博客部署到自己的服务器](https://blog.csdn.net/qq_35561857/article/details/81590953)
* [搭建Git服务器](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)
* [将Hexo博客部署到云主机](https://yq.aliyun.com/articles/640997)
* [Linux系统搭建Git服务器，添加用户名密码实现多用户管理](https://blog.csdn.net/u010258933/article/details/80663805)

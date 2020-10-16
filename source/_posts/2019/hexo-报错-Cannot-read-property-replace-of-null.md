---
title: hexo 报错 Cannot read property 'replace' of null
permalink: hexo-报错-Cannot-read-property-replace-of-null
date: 2019-08-12 19:41:24
tags:
 - Hexo
categories:
 - Hexo
toc: true
---



hexo配置文件进行相应的修改

```
deploy:
  type: git
  repo: https://github.com/qxqzx/qxqzx.github.io.git
  branch: master
```

执行命令

```
hexo g -d
```

就报错了：

```
FATAL Cannot read property 'replace' of null
```

解决：

> 看帖子都是说 _config.yml 配置文件中的url 设置错误

我的设置：

```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' url: 
root: /
permalink: :year/:month/:day/:title/
permalink: hexo-报错-Cannot-read-property-replace-of-null
permalink_defaults:
```

看了下url果然有错，加上url后好了



执行 hexo g -d 成功。

##### 参考链接

[hexo issues #2006](https://link.jianshu.com?t=https://github.com/hexojs/hexo/issues/2006)
 [hexo issues #2141](https://link.jianshu.com?t=https://github.com/hexojs/hexo/issues/2141)
 [cd2want](https://link.jianshu.com?t=http://www.cd2want.cc/2016/06/03/2016-06-03-从jekyll搬到Hexo/)

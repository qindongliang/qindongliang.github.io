# 地址

 [我的Blog预览](http://8090nixi.com/) 欢迎添加友链

 [github 地址](https://github.com/qindongliang/qindongliang.github.io) 欢迎star

# 说明

一款jekyll主题，简洁纯净，支持自适应，感谢tmaize同学的模块。



 
# 截图

![s1](readme/01.jpg)

![s2](readme/02.jpg)

# 使用

## 配置说明

```
encoding: utf-8

# seo
title: TMaize'Blog
description: TMaize'Blog
keywords: TMaize,Blog,TMaize'Blog
author: TMaize

# 上下文环境，"","/blog"
context: ""
scheme: "http"
domain: "blog.tmaize.net"

## 文章url前缀
permalink: /posts/:year/:month/:day/:title.html

copyright: 2016
beianhao: "皖ICP备16016174号"

coderay:
  coderay_tab_width: 4
```


## 写文章

符合 jekyll 的使用规范，请参考我的文件放置规则

文章放在_posts目录

文章资源放在posts目录

## 要修改的文件

+ _includes/footer.html

    请删除本页所有的script标签内的东西

    或者把百度和360的seo推送,mta统计代码替换成你自己的

+ pages/chat.html

    请换成别的评论插件
    
    比如:

    [https://github.com/imsun/gitment](https://github.com/imsun/gitment)

    [https://github.com/gitalk/gitalk](https://github.com/gitalk/gitalk)

+ CNAME

    如果支持cname请换成你自己的域名



---
layout: mypost
title: 如何使用Jekyll+GitHub Pages搭建个人博客站点
categories: [Jekyll Gihub Pages]
---


作为一名有情怀的工程师，一般都会通过博客来记录自己的生活，成长，工作心得或者经验，大部分人使用博客都会经历如下几个阶段：

（1）记录在大脑中 （不推荐）

（2）使用XXX云笔记

（3）使用iteye，csdn，51cto，开源中国，博客园，简书，等等

（4）使用个人站点+微信公众号



（一）Jekyll是什么

jekyll是一个静态博客的生成器，它可以用来把我们使用markdown来写好的文章给转换成静态网页html来发布。


（二）Github Pages是什么

Github Pages 是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点。Github Pages 支持 自动利用 Jekyll 生成站点，也同样支持纯 HTML文档。

（三）如何搭建使用

前提条件：

````
git环境
github账户
ruby环境 
````

jekyll底层是使用ruby编写对，所以安装时候需要先安装ruby环境：

在mac先要安装一些软件，这里假设你对git环境已经有了：

````
brew install ruby

gem install jekyll

gem install bundler

gem install jekyll-paginate

gem install jekyll-gist

````

上面对软件安装完毕后，你就可以在github上搜索一个基于jekyll模版对项目，当然你可以从网上搜索任何你喜欢的主题风格，找到之后使用git clone到自己本地：

 ````
 git clone xxx.git  myblog
 
 cd myblog
 
 jekyll server
 
 ````
然后访问http：//localhost：4040端口就可以在本地预览你到博客了

如果你喜欢这个主题，那么你就可以fork到你自己到github中，然后clone下来，修改一些地方，然后就push到自己到仓库中，就可以了，一些git操作命令：
````
git add .
git commit -m "first commit"
git remote add origin https://github.com/alex-my/alex-my.github.io.git
git push -u origin master
````

关于jekyll的博客的目录结构，感兴趣的可以参考官网文档：
http://jekyllcn.com/docs/structure/

我们写的文章一般是在_posts目录里面，它的格式如下：
````
2018-04-11-spark sql大数据量下的调优和实践.md
````
前面是日期，中间是标题，后缀一般是md，看起来比较简洁。

（四）绑定自己到域名

最后说下github里面的项目，进入项目根目录后，点击右上角的Settings配置选项，在里面可以配置自己的站点域名，我这里配置的是我自己的域名，默认情况下一般都是 username.github.io比如我的是：
````
qindongliang.github.io
````
这样看起来有点简陋，那么绑定我们已经有的域名到github pages上呢，非常easy，首先假设我们已经有一个域名了，没有的话可以自己到网上买，然后在自己到静态站点到根目录下，新建以名字为CNAME到文件，里面的内容就是我们的自己的域名，比如我的：
````
8090nixi.com
````
注意这里只需要域名后面的部分即可，不需要把http和www都写进来，然后登陆域名管理中心，我这里是阿里云的找到域名解析部分，添加一条CNAME记录：
````
CNAME @  qindongliang.github.io
````
配置完毕之后，一般10分钟之内就可以生效，如果不出意外，一会就可以通过我们自定义的域名访问我们的静态站点了。

至此，一个属于我们自己独立的个人站点就完成了，使用jekyll+github pages优缺点如下，借用阮一峰老师的总结：

优点：
````
　  * 免费，无限流量。

　　* 享受git的版本管理功能，不用担心文章遗失。

　　* 你只要用自己喜欢的编辑器写文章就可以了，其他事情一概不用操心，都由github处理。
````
缺点：
````
　　* 有一定技术门槛，你必须要懂一点git和网页开发。

　　* 它生成的是静态网页，添加动态功能必须使用外部服务，比如评论功能就只能用disqus。

　　* 它不适合大型网站，因为没有用到数据库，每运行一次都必须遍历全部的文本文件，网站越大，生成时间越长。
````

但对于中小站点来说，无疑是一个不错的方案，感兴趣的朋友可以尝试一下，我个人比较喜欢简洁的站点风格，大家可以通过我公众号底部的菜单栏的博客按钮感受一下。
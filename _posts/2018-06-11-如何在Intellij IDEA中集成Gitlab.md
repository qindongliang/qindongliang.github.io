---
layout: mypost
title: 如何在Intellij IDEA中集成Gitlab
categories: [IDE]
---


据说在微软收购github当天，一大批用户纷纷转向了gitlab和bitbucket，这两者也都是比较不错的代码托管网站，针对个人和企业都有对应的免费和收费版本，国内公司使用gitlab的应该比较多，而bitbucket比较倾向于个人的私有项目，国内用的人比较少，大家只需要简单了解即可。

今天来简单说下，如何在IDEA中集成gitlab项目，默认情况下IDEA中的 VCS => Checkout From Version Control 选项中是没有gitlab这一项的。

这个时候是没办法直接从IDEA中拉取gitlab里面的项目的，如果想要在IDE中使用，那么需要先把gitlab的分支的项目通过git的clone命令克隆到本地，然后再在IDEA中使用File => Open 命令打开这个项目之后就可以正常操作了，这种方式是最通用的一种办法，就是有点繁琐。

下面看下如何直接从IDEA里面拉取gitlab里面的项目：

（1）在File => Settings => Plugins 里面 搜索 gitlab

![02.jpg](https://ask.qcloudimg.com/draft/1903727/yw9zbjjhz.jpg)

（2）安装这个插件

（3）重启IDEA，再次点击菜单栏 VCS => Checkout From Version Control ，就会发现这次已经有了gitlab选项

![01.jpg](https://ask.qcloudimg.com/draft/1903727/kwz84ju1jw.jpg)

（4）确认安装成功之后，开始配置gitlab

点击File => Settings => Other Settings => GitLab Setting

这里面主要配置GitLab Server Url和你个人的私有访问token，如下：

![03.jpg](https://ask.qcloudimg.com/draft/1903727/uun89zeshn.jpg)

这里说下GitLab Server Url是你们公司或者个人搭建的的首页域名或者ip地址

私有的token，需要你登录到gitlab上，先点击左侧：Profile Settings

![04.jpg](https://ask.qcloudimg.com/draft/1903727/bz33i5n2ot.jpg)

然后点击Account，就能在右侧看到我们的私有token，把这个拷贝上IDEA里面：

![05.jpg](https://ask.qcloudimg.com/draft/1903727/whc8hxmv3f.jpg)

（5）至此，配置已经完成，然后我们就可以在直接在菜单栏中VCS => Checkout From Version Control => GitLab中，看我们的

代码目录：

![06.jpg](https://ask.qcloudimg.com/draft/1903727/fzmaf9p1bf.jpg)

（6）最后，我们随便选择一个项目，打开可以看到有两种check方式，分别是基于SSH和HTTP的，这里大家可以根据情况选择，通常情况下使用HTTP的比较多。

![07.jpg](https://ask.qcloudimg.com/draft/1903727/w17cfi87hf.jpg)

至此，我们已经成功的安装集成完毕。

总结：

同理在JetBrains公司其他的IDE产品中，安装和使用这个插件的思路都一样，如Python的PyCharm中，在使用之前一定先要确定你的机器已经安装过Git，如果没有安装是不能直接使用的，这一点需要注意。

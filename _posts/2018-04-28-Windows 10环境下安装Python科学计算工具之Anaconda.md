---
layout: mypost
title: Windows 10环境下安装Python科学计算工具之Anaconda
categories: [Python]
---



### Anaconda介绍

Anaconda是python加强的一个全家桶套件，是目前最简单的方式来使用python进行机器学习和数据分析，它包含了250多个最流行的python科学计算包，并支持多种系统如windows，linux，mac，此外Anaconda最棒的一个特性就是使用conda来致力于简化包的管理和部署与pip命令的功能类似但更加强大。


### Anaconda下载

Anaconda截止到目前最新的版本是基于Python3.6的Anaconda3 5.1.0，并分别提供了支持Python3.x和Pyhon2.x的发行包，不过建议大家下载使用Python3.x的Anaconda包，因为到2020年Python2.x的工程就不再维护了。


Anaconda已经内置了最新版本的Python3.6，所以大家在windows上安装的时候，不需要提前安装Python，所有的一切Anaconda都已经集成好了，Anaconda内部同时支持Python的2.x和3.x的分支，使用的时候只需要配置相应的解释器即可。

最新版本包的下载地址：

````
https://www.anaconda.com/download/#windows
````

历史其他版本的下载地址：
````
https://repo.continuum.io/archive/
````

### Anaconda安装

这里介绍的是在windows的安装，第二步我们下载好了Anaconda的安装包，在windows上只需要以管理员的身份运行安装即可，安装的目录可以自己设置，

安装完成之后，在windows 10上显示如下：

![image](https://note.youdao.com/yws/public/resource/19321879db39d4dedea8c59d8a9583a3/xmlnote/AB41113E42EB4DE78E39BDF1D90EDEFA/47559)

下面分别介绍下几个组件的功能：


（1）Anaconda Navigator

提供了一个桌面的GUI窗口，，允许你启动应用程序和简单的管理conda包，各种环境而不用使用命令行。



![image](https://note.youdao.com/yws/public/resource/19321879db39d4dedea8c59d8a9583a3/xmlnote/82D84F16AD1E4D1E8A8CA877E4DFF75A/47576)


（2）Anaconda Prompt

提供了一个命令行的交互窗口，安装，升级，卸载，更新python有关的包都可以在这里面进行，不需要再到windows的cmd里面命令。

![image](https://note.youdao.com/yws/public/resource/19321879db39d4dedea8c59d8a9583a3/xmlnote/DA1EC3B295114FBF80BE6AA5E602CC5E/47588)



（3）Jupyter Notebook

直接点击打开，或在终端中输入： jupyter notebook 以启动服务器；在浏览器中打开notebook页面地址：http://localhost:8888 。Jupyter Notebook是一种 Web 应用，能让用户将说明文本、数学方程、代码和可视化内容全部组合到一个易于共享的文档中。

![image](https://note.youdao.com/yws/public/resource/19321879db39d4dedea8c59d8a9583a3/xmlnote/2767EB58A2D4438F862BE188DC2014A8/47595)

（4）Spyder

Spyder是一个使用Python语言的开放源代码跨平台科学运算IDE。Spyder集成了NumPy，SciPy，Matplotlib与IPython，以及其他开源软件，Anaconda内置了Spyder，我们直接安装好Anaconda完毕之后，就可以直接使用这个IDE，当然我喜欢用JetBrains公司的Pycharm。


![image](https://note.youdao.com/yws/public/resource/2e9cccee49686147ada3f77cc3f0f3ec/xmlnote/59FBF8A091084AF581AEB4D0DC46F0F5/47730)

（5）Reset Spyder Settings

这个就不用说了，重置Spyder的配置



### Anaconda与Pycharm集成


JetBrains公司出了很多不错的IDE，比如Java界常用的IDEA，PHP的PhpStorm，Web的WebStorm，C++和C的Rider ，Ruby的RubyMine等等，那么python的就是Pycharm。

下载地址如下：

<https://www.jetbrains.com/pycharm/download/download-thanks.html?platform=windows>


注意PyCharm是需要收费的，不差钱的就购买正版的lincese授权，如果不想买就网上找一些破解的lincese序号。


此外，注意安装Pycharm需要依赖Java的JRE环境，大家可以上Oracle的官网下载，如果是Java开发者，已经安装好了JDK环境，那么就不需要考虑这个问题了。

安装完成之后，配置一下python的解释器即可，如下：

![image](https://note.youdao.com/yws/public/resource/2e9cccee49686147ada3f77cc3f0f3ec/xmlnote/BB70C06A4D51475FBE39A3FA05485A91/47732)


至此集成完毕，大家可以新建一个project，然后直接写python代码，如果需要类库支持，可以直接使用conda安装完毕后，pycharm里面就能够自动识别到，非常方便。



### Anaconda的conda一些命令介绍

````
安装包管理，

列出已经安装的包：在命令提示符中输入pip list或者用conda list
安装新包：在命令提示符中输入“pip install 包名”，或者“conda install 包名”
更新包： conda update package_name
升级所有包： conda upgrade --all
卸载包：conda remove package_names
搜索包：conda search search_term
管理环境：

安装nb_conda，用于notebook自动关联nb_conda的环境
创建环境：在Anaconda终端中 conda create -n env_name package_names[=ver]
使用环境：在Anaconda终端中 activate env_name
离开环境：在Anaconda终端中 deactivate
导出环境设置：conda env export > environmentName.yaml 或 pip freeze > environmentName.txt
导入环境设置：conda env update -f=/path/environmentName.yaml 或 pip install -r /path/environmentName.txt
列出环境清单：conda env list
删除环境： conda env remove -n env_name
````


### 可能遇到的问题

大部分安装完毕后，可能会发现除了Anaconda Prompt能以管理员的命令打开，此外其他的几个组件都打不开，如果这个时候你在pycharm里面使用matplotlib里面使用plt命令打开一个窗口发现控制台没任何报错信息，就是打不开，如果你遇到了类似的问题，很有可能是pyqt的GUI版本太低导致的，可以尝试下面的方法：


（1）先把conda所有的依赖包升级一遍

在Anaconda Prompt窗口里面执行下面的命令
````
conda upgrade --all
````

如果没有解决就进入到第二个步骤

（2）使用pip强制升级qypt5

````
pip install -U qypt5
````

升级完成之后，上述所有的问题都可以完美解决



### 总结

本文主要介绍了Anaconda是什么及win上环境下如何下载，安装和使用，并介绍了其与pycharm的集成方法，最后列举了一些win 10环境上可能出现的问题及解决方法，如果你正准备使用python进行大数据分析，机器学习，计算机图像处理和数据挖掘，那么Anaconda无疑是你最好的选择，没有之一。

---
layout: mypost
title: windows 10环境下安装Tesseract-OCR与python集成
categories: [Python]
---



###  前言
Tesseract是一个开源的ocr引擎，可以开箱即用，项目最初由惠普实验室支持，1996年被移植到Windows上，1998年进行了C++化。在2005年Tesseract由惠普公司宣布开源。2006年到现在，都由Google公司开发。



官网宣传目前支持100多种语言的识别，根据我的测试，目前感觉其对机器打印的比较规整的英语，或者阿拉伯数字的识别准确率还是挺高的，但是对手写的任何东西，效果都非常一般，不过这已经相当不错了。


### 环境介绍

基础软件介绍：

```
windows 10
anaconda 4.5.4
python 3.6.5
opencv 3.4.1 (非必须)
pycharm 2018 (非必须，可以用自己爱好的ide)
```

注意这里我直接装的anaconda4.x（一个python的科学管理软件与java的maven比较类似）的版本，它已经内置支持python的各种版本，省去了一些兼容问题，同时在anaconda的cmd窗口中，如果不想使用自身的conda命令安装软件，我们还可以用pip命令安装，这一点是不冲突的，关于anaconda的安装请参考我前面的文章。


### Tesseract的安装

Tesseract的github地址：<https://github.com/tesseract-ocr/tesseract>

Tesseract的安装：

（1）Tesseract本身没有windows的安装包，不过它指定了一个第三方的封装的windows安装包，在其wiki上有说明，大家可直接到这个地址进行下载：
<https://digi.bib.uni-mannheim.de/tesseract/>

下载后就是一个exe安装包，直接右击安装即可，安装完成之后，配置一下环境变量，编辑 系统变量里面 path，添加下面的安装路径：
```
C:\Program Files (x86)\Tesseract-OCR
```

安装完成之后，直接cmd输入：
```
命令：
tesseract -v
输出如下，即代表成功：
tesseract 4.0.0-beta.1-108-gf291
 leptonica-1.76.0
  libgif 5.1.4 : libjpeg 8d (libjpeg-turbo 1.5.3) : libpng 1.6.34 : libtiff 4.0.9 : zlib 1.2.11 : libwebp 0.6.1 : libopenjp2 2.2.0

```
注意，这一步在windows上是必须安装的，否则运行程序时，会抛出异常：

```
[WinError 2] 系统找不到指定的文件
```

（2）安装python的封装接口：
```
pip install pillow  #一个python的图像处理库，pytesseract依赖
pip install pytesseract
```

注意第一步必须安装成功，同时配置好环境变量，否则第二步必会报错，因为第二步是接口，运行时候会调用第一步的原C++写的类库。



### Tesseract的使用

测试图1，纯数字：

![image](https://ask.qcloudimg.com/draft/1903727/thjnbshlw2.jpg?imageView2/0/w/1620)

结果：
```
140378
```

测试图2，英文：

![image](https://ask.qcloudimg.com/draft/1903727/1kfhiznu7c.jpg?imageView2/0/w/1620)

结果：

```
As you can see in this screenshot, the thresholded image is very clear and the background
has been removed. Our script correctly prints the contents of the image to the console.
```

测试图3，手写数字：

![image](https://ask.qcloudimg.com/draft/1903727/w88oq8c8xv.jpg?imageView2/0/w/1620)

结果：
```
ar oe
```


python代码如下：
```
from  PIL import  Image
import pytesseract
import  cv2 as cv


img_path='F:/fb/xxx.jpg'

# img_path='orgin.jpg'

# img_path='F:/fb/hpop.jpg'

# 依赖opencv
img=cv.imread(img_path)
text=pytesseract.image_to_string(Image.fromarray(img))


# 不依赖opencv写法
# text=pytesseract.image_to_string(Image.open(img_path))


print(text)


```


前面说过，对于机器打印的比较规则的字符，Tesseract识别起来还是比较给力的，至于手写的字符，识别效果比较差，可以看到上面的手写数字识别出来的都是错误的，当然这里也有调优的余地，比如给图片做灰度，模糊，去燥，二值化等等，可能结果会稍微好一点。


### 总结

本篇文章介绍了Tesseract在windows环境下的安装配置，同时介绍了如何在python中集成使用，感兴趣的朋友可以尝试一下。





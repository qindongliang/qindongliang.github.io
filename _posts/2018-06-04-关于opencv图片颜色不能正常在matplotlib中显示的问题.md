---
layout: mypost
title: 关于opencv图片颜色不能正常在matplotlib中显示的问题
categories: [Opencv]
---




opencv默认的彩色图片的加载方式是按照BGR加载的，直接用opencv的函数展示是没有问题的，但是有时候我们想把多张图片放在一起展示，这时候用matplotlib就比较方便，但是matplotlib的图片展示是按照RGB展示的，如果中间不处理一下，直接展示opencv加载的图片，你会发现图片的颜色会出现问题，如何解决？


比较简单，使用opencv的函数把彩色图片转成RGB模式后，再用matplotlib展示就可以了。


效果如下：


![image](https://note.youdao.com/yws/public/resource/850fddba761d4d09d118dc67b7e5d053/xmlnote/149E0155CA024357BBD85FE896E394C3/49930)



上图中左边是BGR的显示模式，后面转成RGB后正常显示，这一点需要用的时候注意下。


源码如下：

```python
# -*- coding:utf-8 -*-
import matplotlib.pyplot as plt
import cv2 as cv
import numpy as np


# 加载原图,彩色的，默认是BGR
img=cv.imread("imgs/22.png")

#  用于存储所有弹框的图片集合
psw=[]

# 转成RGB模式，否则plot不能正常识别
color_img=cv.cvtColor(img,cv.COLOR_BGR2RGB)

# 放入集合
psw.append(("BGR_SHOW",img))
psw.append(("RGB_SHOW",color_img))



# 获取个数
plot_number=len(psw)


# 设置每列显示的窗体个数
cols=2
# 行数自动推算
rows=plot_number/cols+1


# 打印所有的图片
for index in range(plot_number):
    plt.subplot(rows, cols, index + 1)
    plt.imshow(psw[index][1], cmap = "gray")
    plt.title(psw[index][0], fontsize=8)
    plt.xticks([]), plt.yticks([])  # 隐藏坐标轴

plt.show()

```


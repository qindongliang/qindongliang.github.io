---
layout: mypost
title: 如何使用opencv和matplotlib把多个图片显示在一个窗体内
categories: [Opencv]
---

在使用opencv处理一些计算机视觉方面的一些东西时，经常会遇到把多张图片放在一个窗体内对比展示，而不是同时打开多个窗体，opencv作为一个专业的科学计算库，虽然也提供了方法，但使用起来并不是特别灵活而matplotlib作为一个专业的图形库则弥补了这个缺点，下面我们来看下使用。

### 使用opencv展示多张图片

```python
def opecv_muti_pic():
    # 图1
    img = cv.imread('E:\\tmp\\cat.jpg')
    # 图2
    img2 = cv.imread('E:\\tmp\\cat.jpg')
    # 图集
    imgs = np.hstack([img,img2])

    # 展示多个
    cv.imshow("mutil_pic", imgs)

    #等待关闭
    cv.waitKey(0)
```

![](https://ask.qcloudimg.com/http-save/1903727/ath5dxbl8t.png)

注意：

虽然opencv也能正常展示多个图片，但是限制比较大，比如说只能同样尺寸大小的图片，颜色通道一样才能放在一起展示，如果你想展示多个不同的图片在一个opencv的窗体里面，目前好像还不行，包括同一个图片，一个彩色，一个灰度图片都不可以放在一个窗体中，基于这个原因我们大多数时候才使用matplotlib来完成这个任务。

### 使用matplotlib展示多张图片

```python
def matplotlib_multi_pic2():

        plt.gcf().canvas.set_window_title('Test')
        plt.gcf().suptitle("multi pic test")

        # img = cv.imread('E:\\tmp\\cat.jpg')
        img = cv.imread('E:\\tmp\\cat.jpg')
        gray=cv.cvtColor(img,cv.COLOR_BGR2GRAY)
        img2 = cv.imread('E:\\tmp\\test6.jpg')
        gray2 = cv.cvtColor(img2,cv.COLOR_BGR2GRAY)

        img3 = cv.imread('E:\\tmp\\hough.jpg')


        #如果总图片个数不超过10，我们还可以用快速的方法
        plt.subplot(321),plt.imshow(img),plt.title("321")
        plt.subplot(322),plt.imshow(gray),plt.title("322")
        plt.subplot(323),plt.imshow(img2),plt.title("323")
        plt.subplot(324),plt.imshow(gray2),plt.title("324")
        plt.subplot(326),plt.imshow(img3),plt.title("326")


        plt.show()
```

另外一种写法：

```python
def matplotlib_multi_pic1():
    for i in range(9):
        img = cv.imread('E:\\tmp\\cat.jpg')
        title="title"+str(i+1)
        #行，列，索引
        plt.subplot(3,3,i+1)
        plt.imshow(img)
        plt.title(title,fontsize=8)
        plt.xticks([])
        plt.yticks([])
    plt.show()
```

![](https://ask.qcloudimg.com/http-save/1903727/d0h7ublbv1.png)

推荐

源码已经上传到我的github中，感兴趣的朋友可以fork学习：

[https://github.com/qindongliang/opecv3-study/tree/master](https://github.com/qindongliang/opecv3-study/tree/master)

参考文档：

[https://matplotlib.org/api/\_as\_gen/matplotlib.pyplot.subplot.html](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.subplot.html)
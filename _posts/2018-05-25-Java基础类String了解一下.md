---
layout: mypost
title: Java基础类String了解一下
categories: [Java]
---


#### NumPy介绍

NumPy的全名为Numeric Python，是一个开源的Python科学计算库，它包括：

（1）一个强大的N维数组对象ndrray；

（2）比较成熟的（广播）函数库；

（3）用于整合C/C++和Fortran代码的工具包；

（4）实用的线性代数、傅里叶变换和随机数生成函数

主要优点：

1.NumPy数组在数值运算方面的效率优于Python提供的list容器。

2.使用NumPy可以在代码中省去很多循环语句，因此其代码比等价的Python代码更为简洁。


#### ndarray常用属性介绍

序号|数组 | 解释|([[1,2,3]])|([[1],[2],[3]])
---|---|---|---|----|
1|shape|tuple方式返回每个维度的size| (1,3)|(3,1)
2|ndim |直接返回这个数组有多少维|2|2
3|dtype |元素的类型|int32|int32
4|size |元素的个数|3|3


#### ndarray常用创建方法

这里只介绍最常用的方法，从python的list或者tuple中转化成ndarray，关于empty, empty_like, zeros, zeros_like, ones, ones_like, full, full_like
这些方法，请参考官网文档。

````python
def test1():
    # 通过python的list来构建numpy array
    list1 = [[1, 2, 3]]
    list2 = [[1], [2], [3]]

    # 通过python的 tuple来构造
    tuple3= [(1,2,3)]

    # 使用array方法构造
    nd1 = np.array(list1)
    nd2 = np.array(list2)
    nd3 = np.array(tuple3)

    show_array_properties(nd1)
    show_array_properties(nd2)
    show_array_properties(nd3)





def show_array_properties(np_array):
    print("-----------------------")
    """
    常用属性介绍
    :param np_array:
    :return:
    """
    print(np_array.shape) # 代表每一个维度元素的个数
    print(np_array.ndim)  # 总共多少维度
    print(np_array.dtype) # 数据类型
    print(np_array.size) # 数组中元素的个数



test1()
````

输出结果：

````
-----------------------
(1, 3)
2
int32
3
-----------------------
(3, 1)
2
int32
3
-----------------------
(1, 3)
2
int32
3
````


#### ndarray常用数组操作

数组索引下标都是从0开始，不在特意强调


（1）常用步长访问

语法：start:stop:step (开始下标，停止下标，步长)

````
a = np.array([[1,2,3],[3,4,5],[4,5,6]])

print(a[0:3:2]) //start:stop:step

// output
[[1 2 3]
 [4 5 6]]
````

（2）使用arange生成数组，并访问元素
````
a = np.arange(10)

print(a) # [0 1 2 3 4 5 6 7 8 9]

b = a[5]
print b
// 5

b=np.arange(1,6,2)

print(b) # [1 3 5]

````

（3）开始到结束

````
import numpy as np
a = np.arange(10)
print a[2:]
//output
[2  3  4  5  6  7  8  9]
````


（4）指定区间
````
import numpy as np
a = np.arange(10)
print a[2:5]
//output
[2  3  4]
````

（5）多维数组的范围访问
````
import numpy as np
a = np.array([[1,2,3],[3,4,5],[4,5,6]])
print a[1:]

//output
[[3 4 5]
 [4 5 6]]

````

（6）多维数组的列访问

注意下面这种访问情况 冒号可以和三个点号相互替换

````
a = np.array([[1,2,3],[3,4,5],[4,5,6]])

print a[...,1]  ==> [2 4 5]  取第二列
等价
print a[:,1]  ==> [2 4 5]

print a[1,...]  ==> [3 4 5]  取第二行
print a[...,1:] ==> 取第一列之后的所有的列
//output
[[2 3]
 [4 5]
 [5 6]]
````


（7）排序

````
a = np.array([[7,2,3],[3,4,5],[4,5,6]])
print(a[:,0])

# 取每个数组里面里面的第一个元素，排序，返回下标
np.argsort(a[:,0])  #升序
[7,3,4]

// np.argsort(-a[:,0]) #降序

#下面这个是按从小到大排序后的索引值
[1,2,0]

# 取出排序后的元数据

print(a[[1,2,0],:]) 等价 print(a[[1,2,0]])

结果：
[[3 4 5]
 [4 5 6]
 [7 2 3]]

#取a数组，前2个元素
print(a[:2,:])

#取a数组，2后面的元素
print(a[2:,:])

````

（8）reshape转化数组





````

list=[1,2,3,4,5,6,7,8]

array2d=np.array(list)

# 转成 4 行  2列  的 2维数组
print(array2d.reshape(4,2))

# [[1 2]
#  [3 4]
#  [5 6]
#  [7 8]]

````


例子代码可到我github上下载：

<https://github.com/qindongliang/opecv3-study>

上面只是大概介绍了实际应用常用的一些方法，想要了解详细的朋友可以参考官网文档：

<http://www.numpy.org/>



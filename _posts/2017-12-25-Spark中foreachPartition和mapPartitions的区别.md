---
layout: mypost
title: Spark中foreachPartition和mapPartitions的区别
categories: [Spark]
---



spark的运算操作有两种类型：分别是Transformation和Action，区别如下：




Transformation：代表的是转化操作就是我们的计算流程，返回是RDD[T]，可以是一个链式的转化，并且是延迟触发的。

Action：代表是一个具体的行为，返回的值非RDD类型，可以一个object，或者是一个数值，也可以为Unit代表无返回值，并且action会立即触发job的执行。




Transformation的官方文档方法集合如下：

````sh
map
filter
flatMap
mapPartitions
mapPartitionsWithIndex
sample
union
intersection
distinct
groupByKey
reduceByKey
aggregateByKey
sortByKey
join
cogroup
cartesian
pipe
coalesce
repartition
repartitionAndSortWithinPartitions

````


Action的官方文档方法集合如下：
````sh
reduce
collect
count
first
take
takeSample
takeOrdered
saveAsTextFile
saveAsSequenceFile
saveAsObjectFile
countByKey
foreach

````

结合日常开发比如常用的count，collect，saveAsTextFile他们都是属于action类型，结果值要么是空，要么是一个数值，或者是object对象。其他的如map，filter返回值都是RDD类型的，所以简单的区分两个不同之处，就可以用返回值是不是RDD[T]类型来辨别。


接着回到正题，我们说下foreachPartition和mapPartitions的分别，细心的朋友可能会发现foreachPartition并没有出现在上面的方法列表中，原因可能是官方文档并只是列举了常用的处理方法，不过这并不影响我们的使用，首先我们按照上面的区分原则来看下foreachPartition应该属于那种操作，官网文档的这个方法api如下：
````java
public void foreachPartition(scala.Function1<scala.collection.Iterator<T>,scala.runtime.BoxedUnit> f)

Applies a function f to each partition of this RDD.

Parameters:
f - (undocumented)
````

从上面的返回值是空可以看出foreachPartition应该属于action运算操作，而mapPartitions是在Transformation中，所以是转化操作，此外在应用场景上区别是mapPartitions可以获取返回值，继续在返回RDD上做其他的操作，而foreachPartition因为没有返回值并且是action操作，所以使用它一般都是在程序末尾比如说要落地数据到存储系统中如mysql，es，或者hbase中，可以用它。


当然在Transformation中也可以落地数据，但是它必须依赖action操作来触发它，因为Transformation操作是延迟执行的，如果没有任何action方法来触发，那么Transformation操作是不会被执行的，这一点需要注意。



一个foreachPartition例子：

````scala
    val sparkConf=new SparkConf()
     val sc=new SparkContext(sparkConf)
      sparkConf.setAppName("spark demo example ")
    val rdd=sc.parallelize(Seq(1,2,3,4,5),3)
    
    rdd.foreachPartition(partiton=>{
      // partiton.size 不能执行这个方法，否则下面的foreach方法里面会没有数据，
      //因为iterator只能被执行一次
      partiton.foreach(line=>{
        //save(line)  落地数据
      })

    })
    
    sc.stop()
````

一个mapPartitions例子：
````scala
     val sparkConf=new SparkConf()
     val sc=new SparkContext(sparkConf)
      sparkConf.setAppName("spark demo example ")
    val rdd=sc.parallelize(Seq(1,2,3,4,5),3)

    rdd.mapPartitions(partiton=>{
      //只能用map，不能用foreach，因为foreach没有返回值
      partiton.map(line=>{
        //save line
      }
      )
    })

    rdd.count()//需要action，来触发执行
    sc.stop()
````


最后，需要注意一点，如果操作是iterator类型，我们是不能在循环外打印这个iterator的size，一旦执行size方法，相当于iterato就会被执行，所以后续的foreach你会发现是空值的，切记iterator迭代器只能被执行一次。





参考文档：

<http://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/rdd/RDD.html>

<https://spark.apache.org/docs/2.1.0/rdd-programming-guide.html>

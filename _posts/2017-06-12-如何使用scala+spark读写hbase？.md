---
layout: mypost
title: 如何使用scala+spark读写hbase？
categories: [Spark  Hbase]
tags: [one two]
---




最近工作有点忙，所以文章更新频率低了点，希望大家可以谅解，好了，言归正传，下面进入今天的主题：

如何使用scala+spark读写Hbase

软件版本如下：
````
scala2.11.8

spark2.1.0

hbase1.2.0
````

公司有一些实时数据处理的项目，存储用的是hbase，提供实时的检索，当然hbase里面存储的数据模型都是简单的，复杂的多维检索的结果是在es里面存储的，公司也正在引入Kylin作为OLAP的数据分析引擎，这块后续有空在研究下。



接着上面说的，hbase存储着一些实时的数据，前两周新需求需要对hbase里面指定表的数据做一次全量的update以满足业务的发展，平时操作hbase都是单条的curd，或者插入一个批量的list，用的都是hbase的java api比较简单，但这次涉及全量update，所以如果再用原来那种单线程的操作api，势必速度回慢上许多。

关于批量操作Hbase，一般我们都会用MapReduce来操作，这样可以大大加快处理效率，原来也写过MR操作Hbase，过程比较繁琐，最近一直在用scala做spark的相关开发，所以就直接使用scala+spark来搞定这件事了，当然底层用的还是Hbase的TableOutputFormat和TableOutputFormat这个和MR是一样的，在spark里面把从hbase里面读取的数据集转成rdd了，然后做一些简单的过滤，转化，最终在把结果写入到hbase里面。


整个流程如下：


（1）全量读取hbase表的数据

（2）做一系列的ETL

（3）把全量数据再写回hbase



核心代码如下：

````scala


//获取conf
 val conf=HBaseConfiguration.create()
  //设置读取的表
  conf.set(TableInputFormat.INPUT_TABLE,tableName)
  //设置写入的表
  conf.set(TableOutputFormat.OUTPUT_TABLE,tableName)
//创建sparkConf    
   val sparkConf=new SparkConf()
   //设置spark的任务名
   sparkConf.setAppName("read and write for hbase ")
   //创建spark上下文
   val sc=new SparkContext(sparkConf)
   
   //为job指定输出格式和输出表名
   
    val newAPIJobConfiguration1 = Job.getInstance(conf)
    newAPIJobConfiguration1.getConfiguration().set(TableOutputFormat.OUTPUT_TABLE, tableName)
    newAPIJobConfiguration1.setOutputFormatClass(classOf[TableOutputFormat[ImmutableBytesWritable]])
   
   
   //全量读取hbase表
      val rdd=sc.newAPIHadoopRDD(conf,classOf[TableInputFormat]
      ,classOf[ImmutableBytesWritable]
      ,classOf[Result]
    )
   
   //过滤空数据，然后对每一个记录做更新，并转换成写入的格式
    val final_rdd= rdd.filter(checkNotEmptyKs).map(forDatas)
    
    //转换后的结果，再次做过滤
    val save_rdd=final_rdd.filter(checkNull)
    
    //最终在写回hbase表
 save_rdd.saveAsNewAPIHadoopDataset(newAPIJobConfiguration1.getConfiguration)
    sc.stop()
 

````

从上面的代码可以看出来，使用spark+scala操作hbase是非常简单的。下面我们看一下，中间用到的几个自定义函数：


第一个：checkNotEmptyKs 

作用：过滤掉空列簇的数据

````scala
  def checkNotEmptyKs(f:((ImmutableBytesWritable,Result))):Boolean={
    val r=f._2
    val rowkey=Bytes.toString(r.getRow)
    val map:scala.collection.mutable.Map[Array[Byte],Array[Byte]]= r.getFamilyMap(Bytes.toBytes("ks")).asScala
    if(map.isEmpty)  false else true
  }

````

第二个：forDatas

作用：读取每一条数据，做update后，在转化成写入操作

````scala

  def forDatas(f: (ImmutableBytesWritable,Result)): (ImmutableBytesWritable,Put)={
      val r=f._2 //获取Result
      val put:Put=new Put(r.getRow) //声明put
      val ks=Bytes.toBytes("ks") //读取指定列簇
      val map:scala.collection.mutable.Map[Array[Byte],Array[Byte]]= r.getFamilyMap(ks).asScala
      map.foreach(kv=>{//遍历每一个rowkey下面的指定列簇的每一列的数据做转化
                val kid= Bytes.toString(kv._1)//知识点id
                var value=Bytes.toString(kv._2)//知识点的value值
		value="修改后的value"
		put.addColumn(ks,kv._1,Bytes.toBytes(value))	//放入put对象
      }
      )
    if(put.isEmpty)  null  else (new ImmutableBytesWritable(),put)

  }

````


第三个：checkNull
作用：过滤最终结果里面的null数据
````scala
  def checkNull(f:((ImmutableBytesWritable,Put))):Boolean={
    if(f==null)  false  else true
  }
````


上面就是整个处理的逻辑了，需要注意的是对hbase里面的无效数据作过滤，跳过无效数据即可，逻辑是比较简单的，代码量也比较少。

除了上面的方式，还有一些开源的框架，也封装了相关的处理逻辑，使得spark操作hbase变得更简洁，有兴趣的朋友可以了解下，github链接如下：


<https://github.com/nerdammer/spark-hbase-connector>

<https://github.com/hortonworks-spark/shc>



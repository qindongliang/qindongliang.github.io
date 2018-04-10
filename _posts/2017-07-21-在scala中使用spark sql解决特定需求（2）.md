---
layout: mypost
title: 在Scala中使用Spark SQL解决特定需求（2）
categories: [Spark SQL]
---


接着上篇文章，本篇来看下如何在scala中完成使用spark sql将不同日期的数据导入不同的es索引里面。



首下看下用到的依赖包有哪些：

````scala

elasticsearch-spark-20_2.11   5.3.2
elasticsearch                 2.3.4
spark-sql_2.11                2.1.0
spark-hive_2.11               2.1.0
spark-core_2.11               2.1.0
hadoop-client                 2.7.3
scala-library                 2.11.8
````


下面看相关的代码，代码可直接在跑在win上的idea中，使用的是local模式，数据是模拟造的：

````scala


import org.apache.spark.sql.types.{DataTypes, StructField}
import org.apache.spark.sql.{Row, SparkSession}//导入Row对象

/**
  * spark sql to es 本地测试例子
  */
object SparkGroupES {


  def main(args: Array[String]): Unit = {

    //构建spark session
    val spark = SparkSession
      .builder().master("local[1]")
      .appName("Spark SQL basic example")
      .config("es.nodes","192.168.10.125").config("es.port","9200")
      .getOrCreate()

    //导入es-spark的包
    import org.elasticsearch.spark.sql._
    import spark.implicits._


    //使用Seq造数据，四列数据
    val df = spark.sparkContext.parallelize(Seq(
      (0,"p1",30.9,"2017-03-04"),
      (0,"u",22.1,"2017-03-05"),
      (1,"r",19.6,"2017-03-04"),
      (2,"cat40",20.7,"2017-03-05"),
      (3,"cat187",27.9,"2017-03-04"),
      (4,"cat183",11.3,"2017-03-06"),
      (5,"cat8",35.6,"2017-03-08"))

     ).toDF("id", "name", "price","dt")//转化df的四列数据s
    //创建表明为pro
    df.createTempView("pro")

    import spark.sql //导入sql函数

    //按照id分组，统计每组数量，统计每组里面最小的价格，然后收集每组里面的数据
    val ds=sql("select dt, count(*) as c ,collect_list(struct(id,name, price)) as res  from pro   group by dt ")
    //需要多次查询的数据，可以缓存起来
    ds.cache()
    //获取查询的结果，遍历获取结果集
    ds.select("dt","c","res").collect().foreach(line=>{
      val dt=line.getAs[String]("dt") //获取日期
      val count=line.getAs[Long]("c")//获取数量
      val value=line.getAs[Seq[Row]]("res")//获取每组内的数据集合，注意是一个Row实体
      println("日期："+dt+" 销售数量： "+count)

      //创建一个schema针对struct结构
      val schema = DataTypes
        .createStructType( Array[StructField](
          DataTypes.createStructField("id", DataTypes.IntegerType, false), //不允许为null
          DataTypes.createStructField("name", DataTypes.StringType, true),
          DataTypes.createStructField("price", DataTypes.DoubleType, true)
        ))
        //将value转化成rdd
        val rdd=spark.sparkContext.makeRDD(value)
        //将rdd注册成DataFrame
        val df =spark.createDataFrame(rdd,schema)
        //保存每一个分组的数据到es索引里面
        EsSparkSQL.saveToEs(df,"spark"+dt+"/spark",Map("es.mapping.id" -> "id"))
//      value.foreach(row=>{//遍历组内数据集合，然后打印
//        println(row.getAs[String]("name")+" "+row.getAs[Double]("price"))
//      })

    })
    println("索引成功")
    spark.stop()
  }

}



````




分析下，代码执行过程：


（1）首先创建了一个SparkSession对象，注意这是新版本的写法，然后加入了es相关配置


（2）导入了隐式转化的es相关的包


（3）通过Seq+Tuple创建了一个DataFrame对象，并注册成一个表


（4）导入spark sql后，执行了一个sql分组查询


（5）获取每一组的数据



（6）处理组内的Struct结构


（7）将组内的Seq[Row]转换为rdd，最终转化为df


（8）执行导入es的方法，按天插入不同的索引里面


（9）结束


需要注意的是必须在执行collect方法后，才能在循环内使用sparkContext，否则会报错的，在服务端是不能使用sparkContext的，只有在Driver端才可以。




---
layout: mypost
title: elasticsearch里面的关于批量读取mget的用法
categories: [elasticsearch]
---

es的api除了提供了基本的curd操作外，还有两个针对批量的操作分别是：

1，批量的读取操作（mget）

2，批量的写入操作（bulk）


本篇文章先介绍mget的用法



Multi Get api 简称（mget）它允许我们一次get大量的document，与get单条数据的api get方法类似，mget查询是基于index，type（可选），id三个条件进行的，比如我们可以一次mget 50条数据，这50条数据可以是在50个不同index中，并且每一个get都可以单独指定它的路由查询信息，或者返回的字段内容。


mget可以批量的根据index，type，id三个字段来获取一批数据，它不能用来查询，最少得需要知道index 和 id两个字段的值，才能进行get，这一点与query是不一样的。


用法如下： 

mget可以有三种请求头 

（1）不指定index 
````sh
GET /_mget 
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
````

（2）指定index

````sh
GET /test/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
````

（3）指定index和type

````sh
GET /test/type/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}

简写方式
GET /test/type/_mget
{
    "ids" : ["1", "2"]
}
````

此外，还可以单独的设置对返回的数据（source）进行过滤操作，默认情况下如果这条数据被store了，那么它会返回整个document。

几种过滤的方式：

使用source过滤
````sh
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}

````

使用fields过滤：

````sh
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
````

source和fields的主要区别在于，source默认将整个json存在一起，在读取时候只需要加载一次然后再解析出来需要的字段，而store字段则是每个字段单独的存储，所以大部分时候推荐使用source字段，虽然会多占一些存储空间，但在读取字段数比较多的情况下，source的性能是比store字段要更好的，但是如果你disable了source字段，则意味着：

（1）你不能够高亮文本（不推荐在服务端做高亮，推荐客户端做）

（2）你不能reindex索引

（3）你不能做partial update

所以综合考虑，推荐还是使用source字段

在get的时候，还可以使用路由字段，如下：
````sh
GET /_mget?routing=key1
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
````


最后在看下在java api里面如何使用：


````
        //构建一个mget的查询
       MultiGetRequestBuilder  multi_get=  client.prepareMultiGet();
        //添加两条get数据
        multi_get.add("a_active","active","1");
        multi_get.add("b_active","active","2","3");

        //获取响应
        MultiGetResponse mgr= multi_get.get();
         //循环读取
        for (MultiGetItemResponse itemResponse : mgr) {

            GetResponse response = itemResponse.getResponse();
            //如果存在则打印响应消息
            if (response.isExists()) {
                String json = response.getSourceAsString();
                System.out.println(" source data: "+json);
            }

        }
````




总结：


本文介绍了es里面的批量读取数据的方法mget，这个方法在日常开发中的使用频度并不是很高，但是在特定场景下会拥有较高的效率，比如上篇文章介绍的es的分布式查询的原理的时候，在第一阶段query从每个shard上查询本地的page数据，然后返回到coordinating节点上，并重新进行全局排序再取指定分页的n条数据，接着到了第二阶段fetch，要把这批数据的内容读取出来返回给client，这个时候就是mget发力的时候，通过id组装成一个mget请求，然后发送到每个shard里面获取结果数据，最终组装后在返回给client，这样一来比单条get的效率要高很多，另外对索引的写入也是如此，下篇文章我们会介绍批量写入bulk的用法




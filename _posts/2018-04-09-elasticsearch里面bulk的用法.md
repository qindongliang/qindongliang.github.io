---
layout: mypost
title: Elasticsearch里面bulk的用法
categories: [Elasticsearch]
---


上篇文章介绍了在es里面批量读取数据的方法mget，本篇我们来看下关于批量写入的方法bulk。




bulk api可以在单个请求中一次执行多个索引或者删除操作，使用这种方式可以极大的提升索引性能。

bulk的语法格式是：
````sh
action and meta_data \n
optional source \n

action and meta_data \n
optional source \n

action and meta_data \n
optional source \n
````


从上面能够看到，两行数据构成了一次操作，第一行是操作类型可以index，create，update，或者delete，第二行就是我们的可选的数据体，使用这种方式批量插入的时候，我们需要设置的它的Content-Type为application/json。


针对不同的操作类型，第二行里面的可选的数据体是不一样的，如下：

````sh
（1）index 和 create  第二行是source数据体
（2）delete 没有第二行
（3）update 第二行可以是partial doc，upsert或者是script
````


我们可以将我们的操作直接写入到一个文本文件中，然后使用curl命令把它发送到服务端：

一个requests文件内容如下：
````sh
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
````

发送命令如下：
````sh
curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
````
响应结果如下：
````sh
{"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"_doc","_id":"1","_version":1,"result":"created","forced_refresh":false}}]}
````

注意由于我们每行必须有一个换行符，所以json格式只能在一行里面而不能使用格式化后的内容，下面看一个正确的post bulk的请求数据体：

````sh
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
````

bulk请求的返回操作的结果也是批量的，每一个action都会有具体的应答体，来告诉你当前action是成功执行还是失败 ：
````js
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 0,
            "_primary_term": 1
         }
      },
      {
         "delete": {
            "_index": "test",
            "_type": "_doc",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404,
            "_seq_no" : 1,
            "_primary_term" : 2
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "_doc",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 2,
            "_primary_term" : 3
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200,
            "_seq_no" : 3,
            "_primary_term" : 4
         }
      }
   ]
}
````

bulk请求的路径有三种和前面的mget的请求类似：

````sh
（1） /_bulk

（2）/{index}/_bulk

（3）/{index}/{type}/_bulk

````

上面的三种格式，如果提供了index和type那么在数据体里面的action就可以不提供，同理提供了index但没有type，那么就需要在数据体里面自己添加type。


此外，还有几个参数可以用来控制一些操作：

（1）数据体里面可以使用_version字段

（2）数据体里面可以使用_routing字段

（3）可以设置wait_for_active_shards参数，数据拷贝到多个shard之后才进行bulk操作

（4）refresh控制多久间隔多搜索可见



最后重点介绍下update操作，update操作在前面的文章也介绍过，es里面提供了多种更新数据的方法如：

````sh
（1）doc
（2）upsert
（3）doc_as_upsert
（4）script
（5）params ，lang ，source
````

在bulk里面的使用update方法和java api里面类似，前面的文章也介绍过详细的使用，现在我们看下在bulk的使用方式：

````sh
POST _bulk
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "_doc", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "_doc", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
````
其实就是非格式化的内容，放在一行然后提交就行了，不同之处在于前面的文章介绍的是单次请求，而使用bulk之后就可以一次请求批量发送多个操作了。



总结：

本篇文章介绍了在es里面bulk操作的用法，使用bulk操作我们可以批量的插入数据来提升写入性能，但针对不同的action的它的数据格式体是不一样的，这一点需要注意，同时在每行数据结束时必须加一个换行符，不然es是不能正确识别其格式的。




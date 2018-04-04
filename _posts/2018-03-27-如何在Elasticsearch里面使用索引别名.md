---
layout: mypost
title: 如何在Elasticsearch里面使用索引别名
categories: [Elasticsearch]
---

在elasticsearch里面给index起一个aliases（别名）能非常优雅的解决两个索引无缝切换的问题，这个功能在某些场景下非常使用。

比如电商的核心商品索引库，除了实时增量数据外，每天都要重建一遍索引，避免index里面的数据和db里面的数据不一致，因为index分shard了，所以要一个一个的shard做全量替换，直到所有的shard替换完毕，才能宣布重建成功。整个过程其实还是风险挺大的，虽然每次只替换一个shard把风险量降到最低，但如果第3个或第4个shard重建有问题，有可能要回滚整个索引，这个问题其实用索引别名的问题就能比较优雅的解决。

旧索引称为a，新索引称为b，他们拥有共同的别名c，而dao层查询的索引名也是c，当新的全量索引b重建完成之后，只需要解除旧索引a与别名c关系，然后添加新索引b与别名c的关系，就能完成无缝切换，中间对用户是无感知的，如果b有问题，那么随时都可以重新解除b的关系并恢复a，这就完成了所谓的回滚操作，非常简单优雅。


在es里面index aliases就像是软连接一样，它可以映射一个或多个索引，提供了非常灵活的特性，使用它我们可以做到：

（1）在一个运行中的es集群中无缝的切换一个索引到另一个索引上

（2）分组多个索引，比如按月创建的索引，我们可以通过别名构造出一个最近3个月的索引

（3）查询一个索引里面的部分数据构成一个类似数据库的视图（views）



es里面操作索引别名的有两个api命令：

````js
_alias 执行单个别名操作

_aliases 原子的执行多个别名操作

````

如何使用？

假设我们有两个索引分别是my_index_v1和my_index_v2现在想通过索引别名来实现无缝切换，他们对外的索引别名叫my_index。

首先我们先创建第一个old index并给你添加aliases
````js
PUT /my_index_v1   //构建索引

PUT /my_index_v1/_alias/my_index   //给索引添加别名
````

创建完成之后，我们可以查询一下他们的关系：

````js
GET /*/_alias/my_index  //查某个别名映射的所有index


GET /my_index_v1/_alias/* //查询某个index拥有的别名
````


返回结果如下： 

````sh
{
    "my_index_v1" : {
        "aliases" : {
            "my_index" : { }
        }
    }
}
````


现在我们构建new index：
````sh
PUT /my_index_v2   //构建索引
````

新索引构建完毕之后，我们就可以执行切换操作命令了：

````js
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
````

上面的操作是顺序的执行的，先接触old index的别名，然后给new index 添加新的别名，这样以来
索引就透明的别切换了，用户不会感觉任何变化，而且也不需要停机操作。


下面看下java api里面如何操作：


（1）添加别名
````java
  client.admin().indices().prepareAliases().addAlias("my_index_v1","my_index");
````

（2）移除别名
````java
        client.admin().indices().prepareAliases().removeAlias("my_index_v1","my_index");
````

（3）删除一个别名后再添加一个
````java
client.admin().indices().prepareAliases().removeAlias("my_index_v1","my_index")
                .addAlias("my_index_v2","my_index").execute().actionGet();
````


当别名添加完毕后，我们在删除，搜索，更新都可以直接使用：

````java
 SearchRequestBuilder search=client.prepareSearch("my_index");
````

有一点需要注意使用别名后，type类型的值不需要在填写，如果你填写了es是会抛异常的，因为它认为你这别名是一个新的索引，所以我们只写index name即可，es服务端知道它的类型。


总结：

本文介绍了es里面别名的功能和作用并讲解了如何使用别名，如果我们的索引不确定未来如何使用时，给索引加一个别名是一个不错的选择。






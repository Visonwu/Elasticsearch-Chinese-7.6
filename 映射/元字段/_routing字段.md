# _routing字段

使用以下公式将文档路由到索引中的特定分片：

`shard_num = hash(_routing) % num_primary_shards`

\_routing的默认值是文档的_id。

可以通过为每个文档指定自定义路由值来实现自定义路由模式。 例如：

```json
PUT my_index/_doc/1?routing=user1&refresh=true 	//1
{
  "title": "This is a document"
}

GET my_index/_doc/1?routing=user1 				//2
```

> //1: 本文档使用user1作为其路由值，而不是其ID。
>
>
> //2: 获取，删除或更新文档时，需要提供相同的路由值。



_routing字段的值可在查询中访问：

```json
GET my_index/_search
{
  "query": {
    "terms": {
      "_routing": [ "user1" ] 			//1
    }
  }
}
```

> //1:在_routing字段上查询（另请参阅 [`ids` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-ids-query.html)）



## 1.使用自定义路由搜索
自定义路由可以减少搜索的影响。 无需将搜索请求散发到索引中的所有分片，而是可以将请求仅发送到与特定路由值（或多个值）匹配的分片：

```json
GET my_index/_search?routing=user1,user2 		//1
{
  "query": {
    "match": {
      "title": "document"
    }
  }
}
```

> //1:该搜索请求将仅在与user1和user2路由值关联的分片上执行。



## 2.使路由值成为必需
使用自定义路由时，重要的是在 [indexing](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-index_.html), [getting](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-get.html), [deleting](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-delete.html), or [updating](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-update.html) 文档时提供路由值。

忘记路由值可能导致文档在多个分片上建立索引。 为了安全起见，可以将_routing字段配置为产生所有CRUD操作所需的自定义路由值：

```json
PUT my_index2
{
  "mappings": {
    "_routing": {
      "required": true 				//1
    }
  }
}

PUT my_index2/_doc/1 				//2
{
  "text": "No routing value provided"
}
```

> //1: 所有文档都需要路由。
>
>
> //2: 此索引请求将引发routing_missing_exception。



## 3.具有自定义路由的唯一ID
当索引指定自定义\_routing的文档时，不能保证\_id在索引中所有分片上的唯一性。 实际上，如果使用不同的_routing值索引，则具有相同_id的文档可能会以不同的分片结尾。

用户必须确保ID在整个索引中是唯一的。



## 4.路由到索引分区
可以配置索引，以便自定义路由值将进入分片的子集，而不是单个分片。 这有助于减轻最终出现集群不平衡的风险，同时仍然可以减少搜索的影响。

这是通过在创建索引时提供索引级别设置index.routing_partition_size来完成的。 随着分区大小的增加，数据分布变得更加均匀，其代价是必须在每个请求中搜索更多的分片。

如果存在此设置，则计算分片的公式将变为：

```word
shard_num = (hash(_routing) + hash(_id) % routing_partition_size) % num_primary_shards
```

也就是说，\_routing字段用于计算索引内的一组分片，然后_id用于选择该组内的一个分片。

要启用此功能，index.routing_partition_size的值应大于1且小于index.number_of_shards。

 

启用后，分区索引将具有以下限制：

- 不能在其中创建具有连接字段关系的映射。
- 索引内的所有映射都必须将_routing字段标记为必填项。












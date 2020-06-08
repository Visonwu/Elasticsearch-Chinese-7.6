# _index 字段

当跨多个索引执行查询时，有时希望添加仅与某些索引的文档关联的查询子句。 _index字段允许对文档建立索引的索引进行匹配。 在某些查询和聚合中以及在排序或编写脚本时，可以访问其值：

```json
PUT index_1/_doc/1
{
  "text": "Document in index 1"
}

PUT index_2/_doc/2?refresh=true
{
  "text": "Document in index 2"
}

GET index_1,index_2/_search
{
  "query": {
    "terms": {
      "_index": ["index_1", "index_2"] 		//1
    }
  },
  "aggs": {
    "indices": {
      "terms": {
        "field": "_index", 					//2
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_index": { 							//3
        "order": "asc"
      }
    }
  ],
  "script_fields": {
    "index_name": {
      "script": {
        "lang": "painless",
        "source": "doc['_index']" 			//4
      }
    }
  }
}
```

> //1： 在_index字段上查询
>
>
> //2: 汇总_index字段
>
>
> //3: 在_index字段上排序
>
>
> //4: 访问脚本中的_index字段

\_index字段实际上是公开的-不会将它作为真实字段添加到Lucene索引中。 这意味着您可以在术语查询或术语查询（或重写为术语查询的任何查询，例如match，query_string或simple_query_string查询）中使用_index字段，以及前缀和通配符查询。 但是，它不支持正则表达式和模糊查询。

_index字段上的查询除了具体的索引名称外还接受索引别名。



> 注：
>
> 指定远程索引名称（例如cluster_1：index_3）时，查询必须包含分隔符“：”。 例如，在cluster _ *：index_3上的通配符查询将匹配远程索引中的文档。 但是，对cluster * index_1的查询仅与本地索引匹配，因为没有分隔符。 此行为符合远程索引名称的常规解析规则。
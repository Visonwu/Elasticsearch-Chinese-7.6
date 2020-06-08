# _id 字段

每个文档都有一个唯一标识它的_id，该索引已建立索引，以便可以使用[GET API](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-get.html) 或者 [`ids` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-ids-query.html).查询来查找文档。

_id字段的值可在某些查询（`term`, `terms`, `match`, `query_string`, `simple_query_string`）中访问。

```json
# Example documents
PUT my_index/_doc/1
{
  "text": "Document with ID 1"
}

PUT my_index/_doc/2?refresh=true
{
  "text": "Document with ID 2"
}

GET my_index/_search
{
  "query": {
    "terms": {
      "_id": [ "1", "2" ] 		//1
    }
  }
}
```

> 在_id字段上查询（另请参见[`ids` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-ids-query.html)）

\_id字段的值也可以通过聚合或排序进行访问，但是不建议这样做，因为它需要在内存中加载大量数据。 如果需要对_id字段进行排序或聚合，建议在另一个启用了doc_values的字段中复制_id字段的内容。



> 注：_id的大小限制为512个字节，较大的值将被拒绝。


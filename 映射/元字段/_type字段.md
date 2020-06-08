# Type 字段

> 在6.0.0中已弃用。
> 请参阅[*Removal of mapping types*](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/removal-of-types.html)

索引的每个文档都与_type和_id相关联。 索引_type字段以便快速按类型名称搜索。

_type字段的值可在查询，聚合，脚本以及排序时访问：

```json
# Example documents

PUT my_index/_doc/1?refresh=true
{
  "text": "Document with type 'doc'"
}

GET my_index/_search
{
  "query": {
    "term": {
      "_type": "_doc"  		//1
    }
  },
  "aggs": {
    "types": {
      "terms": {
        "field": "_type", 	//2
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_type": { 			//3
        "order": "desc"
      }
    }
  ],
  "script_fields": {
    "type": {
      "script": {
        "lang": "painless",
        "source": "doc['_type']" 		//4
      }
    }
  }
}
```

> //1: 在_type字段上查询
>
>
> //2: 汇总_type字段
>
>
> //3: 在_type字段上排序
>
>
> //4: 访问脚本中的_type字段
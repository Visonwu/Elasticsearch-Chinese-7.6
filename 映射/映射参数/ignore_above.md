# ignore_above

长度超过ignore_above设置的字符串将不会被索引或存储。 对于字符串数组，将对每个数组元素分别应用ignore_above，并且长于ignore_above的字符串元素将不会被索引或存储。

> 注：如果启用了_source字段，则所有字符串/数组元素仍将出现在_source字段中，这是Elasticsearch中的默认值。



```json
PUT my_index
{
  "mappings": {
    "properties": {
      "message": {
        "type": "keyword",
        "ignore_above": 20 					//1
      }
    }
  }
}

PUT my_index/_doc/1 					//2
{
  "message": "Syntax error"
}

PUT my_index/_doc/2 					//3
{
  "message": "Syntax error with some long stacktrace"
}

GET my_index/_search 					//4
{
  "aggs": {
    "messages": {
      "terms": {
        "field": "message"
      }
    }
  }
}
```

> //1: 该字段将忽略任何超过20个字符的字符串。
>
>
> //2: 该文档已成功建立索引。
>
>
> //3: 该文档将被索引，但不对消息字段进行索引。
>
>
> //4: 搜索返回两个文档，但是术语聚合中仅存在第一个文档。



> 注：可以使用PUT映射API在现有字段上更新ignore_above设置。



此选项对于防止Lucene的术语字节长度限制32766也很有用。



> 注：ignore_above的值是字符计数，但是Lucene计数字节。 如果您使用带有许多非ASCII字符的UTF-8文本，则可能需要将限制设置为32766/4 = 8191，因为UTF-8字符最多可以占用4个字节


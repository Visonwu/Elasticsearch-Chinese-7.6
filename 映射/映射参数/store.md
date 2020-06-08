# store

默认情况下，对字段值进行索引以使其可搜索，但不存储它们。 这意味着可以查询该字段，但是无法检索原始字段值。

通常这无关紧要。 该字段值已经是_source字段的一部分，默认情况下已存储。 如果您只想检索单个字段或几个字段的值，而不是整个_source的值，则可以通过源过滤来实现。

在某些情况下，存储字段可能很有意义。 例如，如果您有一个带有标题，日期和很大内容字段的文档，则可能只想检索标题和日期，而不必从较大的_source字段中提取这些字段：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "store": true 			//1
      },
      "date": {
        "type": "date",
        "store": true 			//1
      },
      "content": {
        "type": "text"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "title":   "Some short title",
  "date":    "2015-01-01",
  "content": "A very long content field..."
}

GET my_index/_search
{
  "stored_fields": [ "title", "date" ] 	//2
}
```

> //1: 标题和日期字段被存储。
>
>
> //2: 该请求将检索标题和日期字段的值。



> 注意：以数组形式返回的存储字段
> 为了保持一致性，存储的字段始终以数组形式返回，因为无法知道原始字段值是单个值，多个值还是空数组。
>
> 如果需要原始值，则应改为从_source字段检索它。



可以合理地存储字段的另一种情况是针对那些未出现在_source字段（例如 [`copy_to` fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/copy-to.html)）中的字段。




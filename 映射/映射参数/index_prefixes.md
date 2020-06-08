# index_prefixes

index_prefixes参数启用术语前缀的索引编制，以加快前缀搜索的速度。 它接受以下可选设置：

| `min_chars` | 要索引的最小前缀长度。 必须大于0，并且默认为2。该值包含在内。 |
| ----------- | ------------------------------------------------------------ |
| `max_chars` | 要索引的最大前缀长度。 必须小于20，并且默认为5。该值包含在内。 |

本示例使用默认的前缀长度设置创建一个文本字段：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "body_text": {
        "type": "text",
        "index_prefixes": { }    	//1
      }
    }
  }
}
```

> //1:空的设置对象将使用默认的min_chars和max_chars设置



本示例使用自定义前缀长度设置：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "full_name": {
        "type": "text",
        "index_prefixes": {
          "min_chars" : 1,
          "max_chars" : 10
        }
      }
    }
  }
}
```








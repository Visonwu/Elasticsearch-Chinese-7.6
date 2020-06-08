# position_increment_gap

[Analyzed](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html)文本字段将术语 [positions](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-options.html)考虑在内，以便能够支持 [proximity or phrase queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-match-query-phrase.html).。 当为具有多个值的文本字段建立索引时，将在值之间添加“伪”间隙，以防止大多数短语查询在值之间进行匹配。 间隙的大小使用position_increment_gap配置，默认为100。



例如：

```json
PUT my_index/_doc/1
{
    "names": [ "John Abraham", "Lincoln Smith"]
}

GET my_index/_search
{
    "query": {
        "match_phrase": {
            "names": {
                "query": "Abraham Lincoln" 			//1
            }
        }
    }
}

GET my_index/_search
{
    "query": {
        "match_phrase": {
            "names": {
                "query": "Abraham Lincoln",
                "slop": 101 					//2
            }
        }
    }
}
```

> //1:此词组查询与我们的文档完全不符。
>
>
> //2: 该短语查询与我们的文档匹配，即使Abraham和Lincoln处于不同的字符串中，因为slop> position_increment_gap。



可以在映射中指定position_increment_gap。 例如：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "names": {
        "type": "text",
        "position_increment_gap": 0 		//1
      }
    }
  }
}

PUT my_index/_doc/1
{
    "names": [ "John Abraham", "Lincoln Smith"]
}

GET my_index/_search
{
    "query": {
        "match_phrase": {		
            "names": "Abraham Lincoln" 		//2
        }
    }
}
```

> //1: 下一个数组元素中的第一项与前一个数组元素中的最后一项相距0个项。
>
>
> //2: 短语查询匹配我们的文档，这很奇怪，但是它是我们在映射中要求的。






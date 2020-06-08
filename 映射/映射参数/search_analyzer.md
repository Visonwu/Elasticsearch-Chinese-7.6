# search_analyzer

通常，应在索引时间和搜索时间应用同一[analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html) ，以确保查询中的术语与反向索引中的术语具有相同的格式。

但是，有时候在搜索时使用其他分析器是有意义的，例如，使用edge_ngram标记器进行自动完成时。

默认情况下，查询将使用在字段映射中定义的分析器，但是可以使用search_analyzer设置将其覆盖：



```json
PUT my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": { 				//1
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "autocomplete", 		//2
        "search_analyzer": "standard" 		//2
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Quick Brown Fox" 			//3
}

GET my_index/_search
{
  "query": {
    "match": {
      "text": {
        "query": "Quick Br", 		//4	
        "operator": "and"
      }
    }
  }
}
```

> //1: 分析设置以定义自定义自动完成分析器。
>
>
> //2: 文本字段在索引时间使用自动完成分析器，但在搜索时使用标准分析器。
>
>
> //3: 该字段被索引为以下术语：[q，qu，qui，quic，quick，b，br，br，br，brow，brown，brown，f，fo，fox]
>
>
> //4: 该查询将搜索以下两个词：[quick，br]



有关此示例的完整说明，请参见[Index time search-as-you- type](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/_index_time_search_as_you_type.html)。

>  可以使用PUT映射API在现有字段上更新search_analyzer设置。
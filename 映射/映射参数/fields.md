# fields

为不同的目的以不同的方式对同一字段建立索引通常很有用。 这是多字段的目的。 例如，字符串字段可以映射为用于全文搜索的文本字段，也可以映射为用于排序或聚合的关键字字段：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text",
        "fields": {
          "raw": { 							//1
            "type":  "keyword"
          }
        }
      }
    }
  }
}

PUT my_index/_doc/1
{
  "city": "New York"
}

PUT my_index/_doc/2
{
  "city": "York"
}

GET my_index/_search
{
  "query": {
    "match": {
      "city": "york" 						//2
    }
  },
  "sort": {
    "city.raw": "asc" 						//3
  },
  "aggs": {
    "Cities": {
      "terms": {	
        "field": "city.raw" 				//3
      }
    }
  }
}
```

> //1: city.raw字段是city字段的关键字版本。
>
>
> //2: city字段可用于全文搜索。
>
>
> //3: city.raw字段可用于排序和聚合



> 多字段不会更改原始的_source字段。

可以使用PUT映射API将新的多字段添加到现有字段中。



## 1.具有多个分析仪的多字段
多字段的另一个用例是以不同的方式分析同一字段以获得更好的相关性。 例如，我们可以使用 [`standard` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis-standard-analyzer.html)（将文本分解为单词）为字段索引，再使用 [`english` analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis-lang-analyzer.html#english-analyzer)（将词以词根形式为根）进行索引：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "text": { 					//1
        "type": "text",
        "fields": {
          "english": { 				//2
            "type":     "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "text": "quick brown fox" } 		//3

PUT my_index/_doc/2
{ "text": "quick brown foxes" } 	//3

GET my_index/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown foxes",
      "fields": [ 					//4
        "text",
        "text.english"
      ],
      "type": "most_fields" 		//4
    }
  }
}
```

> //1: text字段使用standard分析仪。
>
>
> //2: text.english字段使用english分析器。
>
>
> //3: 为两个文档建立索引，一个使用fox，另一个使用foxs。
>
>
> //4: 同时查询text和text.english字段并合并分数。



text字段在第一个文档中包含术语fox，在第二个文档中包含术语foxes。 text.english字段包含两个文档的fox，因为fox是fox的词根。

查询字符串也由standard分析器针对text字段进行分析，并由english分析器针对text.english字段进行分析。 词干字段允许查询foxes以匹配仅包含foxe的文档。 这使我们可以匹配尽可能多的文档。 通过查询无梗文本字段，我们提高了与foxes完全匹配的文档的相关性得分。
















# Search-as-you-type数据类型

search_as_you_type字段类型是一个类似文本的字段，经过优化可为提供按需输入完成情况的查询提供开箱即用的支持。 它创建了一系列子字段，这些子字段将被分析以索引可被部分与整个索引文本值部分匹配的查询有效匹配的术语。 支持前缀完成（即匹配项从输入的开头开始）和中缀完成（即匹配项在输入中的任何位置）。

将这种类型的字段添加到映射时

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "search_as_you_type"
      }
    }
  }
}
```



这将创建以下字段

| `my_field`               | 按照映射中的配置进行分析。 如果未配置分析器，则使用索引的默认分析器 |
| ------------------------ | ------------------------------------------------------------ |
| `my_field._2gram`        | 用大小为2的带状令牌过滤器包装my_field的分析器                |
| `my_field._3gram`        | 用大小为3的带状令牌过滤器包装my_field的分析器                |
| `my_field._index_prefix` | 用边缘ngram令牌过滤器包装my_field._3gram的分析器             |



可以使用max_shingle_size映射参数配置子字段中的带状疱疹的大小。 默认值为3，此参数的有效值为整数值2-4（含2和4）。 将为从2到max_shingle_size（包括max_shingle_size）的每个瓦片大小创建碎片子字段。 在构造自己的分析器时，my_field._index_prefix子字段将始终使用具有max_shingle_size的带状板子字段中的分析器。

增加max_shingle_size将改善具有更多连续项的查询的匹配度，但代价是较大的索引大小。 默认的max_shingle_size通常应该足够。

当索引文档具有根字段my_field的值时，相同的输入文本将自动使用不同的分析链索引到这些字段中的每个字段中。



```json
PUT my_index/_doc/1?refresh
{
  "my_field": "quick brown fox jump lazy dog"
}
```



服务于“按需搜索”用例的最有效查询方式通常是针对bool_prefix类型的multi_match查询，该查询以根search_as_you_type字段及其带状子字段为目标。 这可以以任何顺序匹配查询词，但是如果文档在带状子字段中按顺序包含这些词，则文档的评分会更高。



```json
GET my_index/_search
{
  "query": {
    "multi_match": {
      "query": "brown f",
      "type": "bool_prefix",
      "fields": [
        "my_field",
        "my_field._2gram",
        "my_field._3gram"
      ]
    }
  }
}
```

```json
{
  "took" : 44,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.8630463,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8630463,
        "_source" : {
          "my_field" : "quick brown fox jump lazy dog"
        }
      }
    ]
  }
}
```

要按顺序搜索与查询词严格匹配的文档，或者使用短语查询的其他属性进行搜索，请在根字段上使用match_phrase_prefix查询。 如果最后一项应完全匹配，而不是作为前缀，则也可以使用match_phrase查询。 使用短语查询可能不如使用match_bool_prefix查询有效。

```json
GET my_index/_search
{
  "query": {
    "match_phrase_prefix": {
      "my_field": "brown f"
    }
  }
}
```



## 1.特定于search_as_you_type字段的参数
下列参数在search_as_you_type字段的映射中被接受，并且特定于此字段类型

  max_shingle_size

- （可选，整数）要创建的最大带状疱尺寸。 有效值为2（含）到4（含）。 默认为3。
- 将为2和该值之间的每个整数创建一个子字段。 例如，值为3会创建两个子字段：my_field._2gram和my_field._3gram

- 更多子字段可启用更具体的查询，但会增加索引大小。



## 2.字段类型的参数作为文本字段
由于search_as_you_type字段的性质类似于文本字段，因此以下参数在映射中被接受，并且在配置文本数据类型的字段时其行为类似于它们的行为。 除非另有说明，否则这些选项以相同的方式配置根字段子字段。



[`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html): 在索引时和搜索时都应用于文本字段的分析器（除非被search_analyzer覆盖）。默认为默认索引分析器或标准分析器。

[`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html): 该字段是否可以搜索？接受true（默认）或false。

[`index_options`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-options.html): 哪些信息应存储在索引中，以进行搜索和突出显示。默认为位置。

[`norms`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/norms.html): 在对查询进行评分时是否应考虑字段长度。接受对或错。此选项配置根字段和带状疱疹子字段，其默认值为true。它不配置前缀子字段，它在此处为假。

[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html): 是否应与_source字段分开存储和检索字段值。接受true或false（默认）。此选项仅配置根字段，而不配置任何子字段。

[`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-analyzer.html): 搜索时应在文本字段上使用的分析器。默认为分析仪设置。

[`search_quote_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html#search-quote-analyzer): 遇到短语时在搜索时应使用的分析器。默认为search_analyzer设置。

[`similarity`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/similarity.html): 应该使用哪种评分算法或相似性。默认为BM25。

[`term_vector`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/term-vector.html): 是否应为该字段存储术语向量。默认为否。此选项配置根字段和带状疱疹子字段，但不配置前缀子字段。



## 3.优化前缀查询
在对根字段或其任何子字段进行前缀查询时，该查询将被重写为._index_prefix子字段上的术语查询。 与在文本字段上进行前缀查询相比，这种方法更有效地进行匹配，因为可以将每个带状疱的特定长度的前缀直接作为._index_prefix子字段中的术语进行索引。

._index_prefix子字段的分析器稍稍修改了生成木瓦的行为，以便还在字段值的末尾索引通常不会生成为木瓦的术语的前缀。 例如，如果将快速棕狐值索引到max_shingle_size为3的search_as_you_type字段中，则棕狐和fox的前缀也被索引到._index_prefix子字段中，即使它们未在._3gram子字段中显示为术语。 这样就可以完成该字段输入中的所有条款。
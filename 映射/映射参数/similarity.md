# similarity

Elasticsearch允许您配置评分算法或每个字段的相似度。 相似性设置提供了一种选择默认BM25以外的相似性算法的简单方法，例如TF / IDF。

相似性对于文本字段最有用，但也可以应用于其他字段类型。

可以通过调整内置相似性的参数来配置自定义相似性。 有关此专家选项的更多详细信息，请参见[similarity module](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-modules-similarity.html).。

无需任何进一步配置即可直接使用的相似之处是：

- BM25
  Okapi BM25算法。 Elasticsearch和Lucene中默认使用的算法。 有关更多信息，请参见[Pluggable Similarity Algorithms](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/pluggable-similarites.html) 。
-   经典
  TF / IDF算法曾经是Elasticsearch和Lucene中的默认算法。 有关更多信息，请参见[Lucene’s Practical Scoring Function](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/practical-scoring-function.html)。
-   布尔值
  一个简单的布尔相似性，当不需要全文排名时使用，分数应仅基于查询词是否匹配。 布尔相似度使术语的得分等于其查询量。

首次创建字段时，可以在字段级别上设置相似性，如下所示：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "default_field": { 			//1
        "type": "text"
      },
      "boolean_sim_field": {
        "type": "text",
        "similarity": "boolean" 	//2
      }
    }
  }
}
```

> //1: default_field使用BM25相似性。
>
>
> //2: boolean_sim_field使用布尔相似度。
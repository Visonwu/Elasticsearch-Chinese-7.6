# Token count 数据类型

类型为token_count的字段实际上是一个整数字段，它接受字符串值，对其进行分析，然后索引字符串中的token数。

例如：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "name": { 				//1
        "type": "text",
        "fields": {
          "length": { 			//2
            "type":     "token_count",
            "analyzer": "standard"
          }
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "name": "John Smith" }

PUT my_index/_doc/2
{ "name": "Rachel Alice Williams" }

GET my_index/_search
{
  "query": {
    "term": {
      "name.length": 3 	//3
    }
  }
}
```

> //1: 名称字段是使用默认标准分析器的文本字段。
>
>
> //2: name.length字段是token_count多字段，它将索引name字段中的token数量。
>
>
> //3: 该查询仅匹配包含Rachel Alice Williams的文档，因为它包含三个toeken。





## 1.token_count字段的参数

token_count字段接受以下参数：

| [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html) | [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis.html)应该用于分析字符串值。 需要。 为了获得最佳性能，请使用不带令牌过滤器的分析仪。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `enable_position_increments`                                 | 指示是否应计算位置增量。 如果您不想对分析器过滤器删除的令牌进行计数（例如[`stop`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis-stop- tokenfilter.html)）。 默认为true。 |
| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/doc-values.html) | 该字段是否应该以column-stride的方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？ 接受“ true”（默认）或“ false”。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html) | 该字段是否可以搜索？ 接受“ true”（默认）和“ false”。         |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/null-value.html) | 接受与字段相同的“类型”的数字值，以替换任何显式的“空”值。 默认为“ null”，这意味着该字段被视为丢失。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html) | 字段值是否应与[`_source`](https_www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-source-field.html)字段分开存储和检索。 接受“ true”或“ false”（默认）。 |






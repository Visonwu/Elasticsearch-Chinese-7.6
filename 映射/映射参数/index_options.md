# index_options

`index_options`参数控制将哪些信息添加到反向索引中以进行搜索和突出显示。

> 注意：index_options参数仅适用于文本字段。 避免将index_options与其他字段数据类型一起使用。

它接受以下值：

**docs**
仅文档编号被索引。 可以回答这个字段中是否存在这个term？
**freqs**
文档编号和词频被索引。 词频用于为重复词评分高于单个词。
  **positions（默认）**
索引文档编号，词频和term位置（或顺序）。 位置可用于[邻近或短语](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-match-query-phrase.html)查询。
  **offsets**
会为文档编号，词频，位置以及开始和结束字符偏移（将term映射回原始字符串）建立索引。 [unified highlighter](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-request-body.html#unified-highlighter)使用偏移量来加速突出显示。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "index_options": "offsets"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Quick brown fox"
}

GET my_index/_search
{
  "query": {
    "match": {
      "text": "brown fox"
    }
  },
  "highlight": {
    "fields": {
      "text": {} 
    }
  }
}
```

> //1. 默认情况下，文本字段将使用postings进行突出显示，因为索引了偏移量。


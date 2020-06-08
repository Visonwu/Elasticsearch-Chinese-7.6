# term_vector

术语向量包含有关分析过程产生的术语的信息，包括：

- term列表。
- 每个术语的位置（或顺序）。
- 起始字符和结束字符偏移量，将术语映射到原始字符串中的源。
- payload（如果可用）—与每个术语位置关联的用户定义的二进制数据。

可以存储这些术语向量，以便可以为特定文档检索它们。

term_vector设置接受：

| `no`                              | 没有术语向量被存储。 （默认）                   |
| --------------------------------- | ----------------------------------------------- |
| `yes`                             | 仅存储该字段中的术语。                          |
| `with_positions`                  | Terms 和 positions 已经被存储                   |
| `with_offsets`                    | Terms 和字符偏移量被存储。                      |
| `with_positions_offsets`          | Terms, positions, 和字符偏移量被存储。          |
| `with_positions_payloads`         | Terms, positions, 和payload被存储。             |
| `with_positions_offsets_payloads` | Terms, positions, offsets and payloads 被存储。 |

快速矢量 高亮 需要with_positions_offsets。  [The term vectors API](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-termvectors.html) 一词可以检索存储的任何内容。



> 设置with_positions_offsets将使字段索引的大小加倍。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "text": {
        "type":        "text",
        "term_vector": "with_positions_offsets"
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
      "text": {} 		//1
    }
  }
}
```

> ​	默认情况下，快速矢量高亮将用于文本字段，因为启用了术语矢量。
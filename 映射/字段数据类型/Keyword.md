# Keyword

用于索引结构化内容（例如ID，电子邮件地址，主机名，状态代码，邮政编码或标签）的字段。

它们通常用于过滤（查找状态已发布的所有博客文章），进行排序和汇总。 关键字字段只能按其确切值进行搜索。

如果您需要索引全文内容（例如电子邮件正文或产品说明），则可能应该使用text字段。

以下是keyword字段的映射示例：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "tags": {
        "type":  "keyword"
      }
    }
  }
}
```

> 小知识：**映射数字标识符**
>
> 并非所有数字数据都应映射为[数字字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)数据类型。 Elasticsearch为[range](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html)查询优化数字字段，例如integer或long。 但是，[keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)字段更适合[`term`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) 和其他[term-level](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html) 的查询。
>
> 范围range中很少使用标识符，例如ISBN或产品ID。 但是，通常使用term-level来检索它们。
>
> 在以下情况下，请考虑将数字标识符映射为关键字：
>
> - 您不打算使用range查询来搜索标识符数据。
> - 快速检索很重要。 keyword字段上的字词查询搜索通常比数字字段上的字词搜索更快。
>
> 
>
> 如果不确定要使用哪个数据，可以使用[multi-field](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)将数据映射为关键字和数字数据类型。



## 1.1 keyword的参数

关键字字段接受以下参数：

| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) | 该字段是否应该以column stride fashion的方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？ 接受“ true”（默认）或“ false”。 |
| [`eager_global_ordinals`](https://www.elastic.co/guide/en/elasticsearch/reference/current/eager-global-ordinals.html) | 是否应在刷新时热切加载全局序号？ 接受“ true”或“ false”（默认）。 在经常用于术语聚合的字段上启用此功能是个好主意。 |
| [`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html) | 多字段允许针对不同目的以多种方式为同一字符串值建立索引，例如一个字段用于搜索，而一个多字段用于排序和聚合。 |
| [`ignore_above`](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-above.html) | 不要索引比该值长的任何字符串。 默认为2147483647，以便接受所有值。 但是请注意，默认动态映射规则会通过设置`ignore_above：256`来创建一个子关键字字段来覆盖此默认值。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-index.html) | 该字段是否可以搜索？ 接受“ true”（默认）或“ false”。         |
| [`index_options`](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-options.html) | 出于评分目的，应该在索引中存储哪些信息。 默认为`docs`，但也可以设置为`freqs`以在计算分数时考虑词频。 |
| [`norms`](https://www.elastic.co/guide/en/elasticsearch/reference/current/norms.html) | 在对查询进行评分时是否应考虑字段长度。 接受“ true”或“ false”（默认）。 |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html) | 接受一个字符串值，该字符串值将替换任何显式的“ null”值。 默认为“ null”，这意味着该字段被视为丢失。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) | 字段值是否应该与[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)字段分开存储和检索。 接受“ true”或“ false”（默认）。 |
| [`similarity`](https://www.elastic.co/guide/en/elasticsearch/reference/current/similarity.html) | 应该使用哪种评分算法或“相似度”。 默认为BM25。                |
| [`normalizer`](https://www.elastic.co/guide/en/elasticsearch/reference/current/normalizer.html) | 如何在索引之前对关键字进行预处理。 默认为“ null”，表示关键字保持原样。 |
| `split_queries_on_whitespace`                                | 在构建对此字段的查询时，[全文查询](https://www.elastic.co/guide/zh/elasticsearch/reference/current/full-text-queries.html)是否应在空白处分割输入。 接受“ true”或“ false”（默认）。 |
| [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-meta.html) | 有关该字段的元数据。                                         |

> 注意：从2.x导入的索引不支持关键字。 相反，他们将尝试将关键字降级为字符串。 这使您可以将现代映射与旧版映射合并。 升级到6.x之前必须重新创建长期索引，但是映射降级使您有机会按自己的时间表进行重新创建。




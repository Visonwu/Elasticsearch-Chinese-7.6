# Text 数据类型

用于索引全文值的字段，例如电子邮件的正文或产品的描述。 对这些字段进行分析，也就是说，将它们通过分析器传递，以在将其编入索引之前将字符串转换为单个术语的列表。 通过分析过程，Elasticsearch可以在每个全文字段中搜索单个单词。 文本字段不用于排序，很少用于聚合（尽管重要的文本聚合是一个明显的例外）。

如果您需要索引结构化的内容（例如电子邮件地址，主机名，状态代码或标签），则可能应该使用关键字字段。

以下是文本字段映射的示例：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "full_name": {
        "type":  "text"
      }
    }
  }
}
```



## 1.使用字段作为文本和关键字
有时，在同一字段中同时具有全文本（文本）和关键字（关键词）版本会很有用：一个用于全文本搜索，另一个用于聚合和排序。 这可以通过[multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/multi-fields.html).来实现。



## 2.文本字段的参数
文本字段接受以下参数：

| [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html) | [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis.html)在索引时间和搜索时间均应用于“文本”字段 （除非被[`search_analyzer`](https://www.elastic.co/guide/zh-CN/elasticsearch/reference/7.6/search-analyzer.html覆盖)）。 默认为默认的索引分析器或[标准分析器](https://www.elastic.co/guide/zh/elasticsearch/reference/7.6/analysis-standard-analyzer.html)。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| [`eager_global_ordinals`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/eager-global-ordinals.html) | 是否应在刷新时热切加载全局序号？ 接受“ true”或“ false”（默认）。 在经常用于（重要）术语聚合的字段上启用此功能是个好主意。 |
| [`fielddata`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/fielddata.html) | 字段使用内存中的字段数据进行排序，聚合或编写脚本？ 接受“ true”或“ false”（默认）。 |
| [`fielddata_frequency_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/fielddata.html#field-data-filtering) | 启用“ fielddata”后，可以确定要加载哪些值的专家设置。 默认情况下，所有值均已加载。 |
| [`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/multi-fields.html) | Multi-fields允许针对不同目的以多种方式为同一字符串值建立索引，例如，一个字段用于搜索，多字段用于排序和聚合，或者由不同的分析器分析同一字符串值。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html) | 该字段是否可以搜索？ 接受“ true”（默认）或“ false”。         |
| [`index_options`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-options.html) | 哪些信息应存储在索引中，以进行搜索和突出显示。 默认为“ positions”。 |
| [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-prefixes.html) | 如果启用，则2到5个字符之间的术语前缀会被索引到一个单独的字段中。 这样可以使前缀搜索更有效地运行，但要以较大的索引为代价。 |
| [`index_phrases`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-phrases.html) | 如果启用，则将两个词的单词组合（* shingles *）索引到一个单独的字段中。 这使得精确的短语查询（无延迟）可以更有效地运行，但要以较大的索引为代价。 请注意，这在不删除停用词的情况下效果最佳，因为包含停用词的短语将不会使用子字段，并且会退回到标准短语查询中。 接受“ true”或“ false”（默认）。 |
| [`norms`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/norms.html) | 在对查询进行评分时是否应考虑字段长度。 接受“ true”（默认）或“ false”。 |
| [`position_increment_gap`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/position-increment-gap.html) | 应当在字符串数组的每个元素之间插入的虚假术语位置的数量。 默认为在分析仪上配置的“ position_increment_gap”，默认为“ 100”。 之所以选择“ 100”，是因为它防止了具有较大斜率（小于100）的词组查询在字段值之间匹配词条。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html) | 字段值是否应与[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-source-field.html)字段分开存储和检索。 接受“ true”或“ false”（默认）。 |
| [`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-analyzer.html) | 在搜索时应在[text]字段上使用的[`analyzer`](https://www.elastic.co/guide/zh/elasticsearch/reference/7.6/analyzer.html)。 默认为`analyzer`设置。 |
| [`search_quote_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analyzer.html#search-quote-analyzer) | 遇到词组时应在搜索时使用的[`analyzer`](https://www.elastic.co/guide/zh/elasticsearch/reference/7.6/analyzer.html)。 默认为`search_analyzer`设置。 |
| [`similarity`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/similarity.html) | 应该使用哪种评分算法或“相似度”。 默认为BM25。                |
| [`term_vector`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/term-vector.html) | 是否应为该字段存储术语向量。 默认为“ no”。                   |
| [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-field-meta.html) | 有关该字段的元数据。                                         |


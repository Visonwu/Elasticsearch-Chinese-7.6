# Flattened 数据类型

默认情况下，对象中的每个子字段都分别进行映射和索引。 如果子字段的名称或类型事先未知，则将[动态映射](<https://www.elastic.co/guide/en/elasticsearch/reference/7.6/dynamic-mapping.html>)它们。

扁平类型提供了一种替代方法，其中将整个对象映射为单个字段。 对于给定的对象，展平的映射将解析出其叶值并将它们作为关键字索引到一个字段中。 然后可以通过简单的查询和汇总来搜索对象的内容。

此数据类型对于索引具有大量或未知数量唯一键的对象很有用。 仅为整个JSON对象创建一个字段映射，这可以帮助防止由于大量不同的字段映射而导致[映射爆炸](<https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping.html#mapping-limit-settings>)。



另一方面，展平的对象字段在搜索功能方面存在折衷。 仅允许基本查询，不支持数字范围查询或突出显示。 有关限制的更多信息，请参见“[支持的操作](<https://www.elastic.co/guide/en/elasticsearch/reference/7.6/flattened.html#supported-operations>)”部分。

扁平化的映射类型不应用于索引所有文档内容，因为它将所有值都视为关键字，并且不提供完整的搜索功能。 在大多数情况下，默认方法（每个子字段在映射中都有其自己的条目）有效。

```json
PUT bug_reports
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "labels": {
        "type": "flattened"
      }
    }
  }
}

POST bug_reports/_doc/1
{
  "title": "Results are not sorted correctly.",
  "labels": {
    "priority": "urgent",
    "release": ["v1.2.5", "v1.3.0"],
    "timestamp": {
      "created": 1541458026,
      "closed": 1541457010
    }
  }
}
```

在索引期间，将为JSON对象中的每个叶值创建令牌。 这些值被索引为字符串关键字，无需分析或对数字或日期进行特殊处理。

查询顶级扁平化字段将搜索对象中的所有叶值：

```json
POST bug_reports/_search
{
  "query": {
    "term": {"labels": "urgent"}
  }
}
```

要查询展平对象中的特定键，请使用对象点表示法：

```json
POST bug_reports/_search
{
  "query": {
    "term": {"labels.release": "v1.3.0"}
  }
}
```



## 1.支持的操作

由于索引值的方式相似，因此，扁平字段与关键字字段共享许多相同的映射和搜索功能。

当前，展平的对象字段可以与以下查询类型一起使用：

> - `term`, `terms`, and `terms_set`
> - `prefix`
> - `range`
> - `match` and `multi_match`
> - `query_string` and `simple_query_string`
> - `exists`



查询时，不可能使用通配符来引用字段关键字，例如`{“ term”：{“ labels.time *”：1541457010}}`。 请注意，所有查询（包括范围）都将值视为字符串关键字。 拼合的字段不支持突出显示。

可以对展平的对象字段进行排序，以及执行简单的关键字样式聚合（例如术语）。 与查询一样，对数字没有特殊支持-将JSON对象中的所有值都视为关键字。 排序时，这意味着按字典顺序对值进行比较。

展平的对象字段当前无法存储。 无法在映射中指定store参数。



## 2.对于Flattened字段的参数

接受以下映射参数：

| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `depth_limit`                                                | 根据嵌套的内部对象，展平的对象字段的最大允许深度。 如果展平的对象字段超出此限制，则将引发错误。 默认为20。 请注意，可以通过[put mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-put-mapping.html)API动态更新“ depth_limit”。 |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/doc-values.html) | 该字段是否应该以column-stride fashion的方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？ 接受“ true”（默认）或“ false”。 |
| [`eager_global_ordinals`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/eager-global-ordinals.html) | 是否应在刷新时热切加载全局序号？ 接受“ true”或“ false”（默认）。 在经常用于术语聚合的字段上启用此功能是个好主意。 |
| [`ignore_above`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/ignore-above.html) | 超过此限制的叶值将不会被索引。 默认情况下，没有限制，所有值都将被索引。 请注意，此限制适用于展平对象字段内的叶子值，而不适用于整个字段的长度。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html) | 确定该字段是否应可搜索。 接受“ true”（默认）或“ false”。     |
| [`index_options`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/index-options.html) | 出于评分目的，应在索引中存储哪些信息。 默认为`docs`，但也可以设置为`freqs`以在计算分数时考虑词频。 |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/null-value.html) | 一个字符串值，用于替换扁平对象字段中的任何显式“ null”值。 默认为“ null”，这意味着将空字段视为缺失。 |
| [`similarity`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/similarity.html) | 应该使用哪种评分算法或“相似度”。 默认为BM25。                |
| `split_queries_on_whitespace`                                | 在构建对此字段的查询时，[全文查询](https://www.elastic.co/guide/zh/elasticsearch/reference/7.6/full-text-queries.html)是否应在空白处分割输入。 接受“ true”或“ false”（默认）。 |
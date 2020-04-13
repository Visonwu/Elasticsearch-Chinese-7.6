# eager_global_ordinals

## 1.1 什么是global ordinals？

为了支持需要在每个文档中查找字段值的聚合和其他操作，Elasticsearch使用称为[doc_values](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/doc-values.html)的数据结构。 基于术语的字段类型（例如关键字）使用ordinal映射存储其doc_value，以实现更紧凑的表示形式。 这种映射的工作方式是根据其词典顺序为每个术语分配一个增量整数或序数。 该字段的doc vlaues仅存储每个文档的序号，而不存储原始术语，并具有单独的查找结构，可在ordinal和术语之间进行转换。

在聚合过程中使用时，通常可以大大提高性能。 例如，术语聚合仅依靠ordinal将文档收集到分片级别的存储桶中，然后在合并各个分片的结果时将ordinal转换回其原始术语值。

每个索引段都定义了自己的序数映射，但是聚合在整个分片上收集数据。 为了能够将序数用于诸如聚合之类的分片级操作，Elasticsearch创建了一个称为全局序数的统一映射。 全局序数映射建立在分段序数之上，并通过维护每个分段从全局序数到局部序数的映射来工作。

如果搜索包含以下任何组件，则使用全局序号：

- keyword，ip和flatted字段上的某些存储桶聚合。 这包括如上所述的术语聚合，以及composite，diversified_sampler和significant_terms。
- 需要启用[fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/fielddata.html)的text字段上的存储桶聚合。
- 连接字段中对父文档和子文档的操作，包括has_child查询和parent聚合。

---待修正

> 注：全局序号 映射是堆上的数据结构。 在测量内存使用情况时，Elasticsearch将来自全局序号的内存作为fielddata进行计数。 全局序号内存被包含在[fielddata circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/circuit-breaker.html#fielddata-circuit-breaker)中，并在[node stats](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cluster-nodes-stats.html)信息响应中的fielddata返回。



## 1.2 加载global ordinals

必须先建立全局序数映射，然后才能在搜索过程中使用序数。 默认情况下，映射是在第一次需要全局序号时在搜索过程中加载的。 如果您正在优化索引速度，那么这是正确的方法，但是如果要优先考虑搜索性能，建议您急切地将全局序号加载到将用于聚合的字段上：

```json
PUT my_index/_mapping
{
  "properties": {
    "tags": {
      "type": "keyword",
      "eager_global_ordinals": true
    }
  }
}
```

启用eager_global_ordinals时，会在刷新分片时构建全局序号。— Elasticsearch始终在暴露索引内容更改之前加载它们。 这将构建全局索引的成本从搜索转移到了索引时间。 当创建新的分片副本时，Elasticsearch也会急于建立全局序号，这可能会在增加副本数量或将分片重新放置到新节点上时发生。

可以随时通过更新eager_global_ordinals设置来禁用预先加载：

```json
PUT my_index/_mapping
{
  "properties": {
    "tags": {
      "type": "keyword",
      "eager_global_ordinals": false
    }
  }
}
```

> 重要：
>
> 在[冻结索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/frozen-indices.html)上，全局序号在每次搜索后都会被丢弃，并在需要时重新构建。 这意味着`eager_global_ordinals`不应用于冻结索引：这将导致在每次搜索时重新加载全局序数。 相反，在冻结之前，应将索引强制合并到单个段。 这样可以避免完全建立全局序号（更多细节可以在下一节中找到）。



## 1.3 避免global ordinal加载

通常，就其加载时间和内存使用而言，全局序号不会带来很大的开销。 但是，对于具有大碎片的索引，或者如果字段包含大量唯一项值，则加载全局序号可能会很昂贵。 因为全局序号为分片上的所有分段提供了统一的映射，所以当新的分段变为可见时，还需要完全重建它们。

在某些情况下，可以完全避免全局序数加载：

- term，sampler和significant_terms聚合支持参数[execution_hint](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-execution-hint)，该参数有助于控制如何收集存储桶。 它默认为global_ordinals，但可以设置为映射以直接使用术语值。
- 如果某个分片已被[fore-merged](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html)为单个片段，则该分片的序号对于该分片已经是全局的。 在这种情况下，Elasticsearch不需要构建全局序数映射，并且使用全局序数不会产生额外的开销。 请注意，出于性能方面的考虑，您只应强制合并一个索引，以后再也不会写入该索引。


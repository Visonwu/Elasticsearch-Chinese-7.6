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

> 注：全局序号 映射是堆上的数据结构。 在测量内存使用情况时，Elasticsearch将来自全局序号的内存作为字段数据进行计数。 全局序号内存被包含在[fielddata circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/circuit-breaker.html#fielddata-circuit-breaker)中，并在[node stats](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cluster-nodes-stats.html)信息响应中的fielddata下返回。

The global ordinal mapping is an on-heap data structure. When measuring memory usage, Elasticsearch counts the memory from global ordinals as *fielddata*. Global ordinals memory is included in the [fielddata circuit breaker](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/circuit-breaker.html#fielddata-circuit-breaker), and is returned under `fielddata` in the [node stats](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/cluster-nodes-stats.html) response.
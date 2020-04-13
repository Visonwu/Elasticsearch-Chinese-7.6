# fielddata

默认情况下，大多数字段都已编入索引，这使它们可搜索。 但是，脚本中的排序，聚合和访问字段值需要与搜索不同的访问模式。

搜索需要回答“哪个文档包含该术语？”这个问题，而排序和汇总则需要回答一个不同的问题：“此字段对于此文档的值是什么？”。

大多数字段可以将索引时间的磁盘doc_values用于此数据访问模式，但是文本字段不支持doc_values。

而是，文本字段使用称为fielddata的查询时内存数据结构。 首次将字段用于聚合，排序或在脚本中使用时，将按需构建此数据结构。 它是通过从磁盘读取每个段的整个反向索引，反转术语↔︎文档关系并将结果存储在JVM堆中的内存中来构建的。



## 1.1 fielddata 在text字段类型模式是不可用的

字段数据会占用大量堆空间，尤其是在加载高基数的文本字段时。 一旦将字段数据加载到堆中，它在该段的生命周期内将一直保留在那里。 同样，加载字段数据是一个昂贵的过程，可能导致用户遇到延迟命中的情况。 这就是默认情况下禁用字段数据的原因。

如果您尝试对text字段中的脚本进行排序，汇总或访问值，则会看到以下异常：

```text
Fielddata is disabled on text fields by default.  Set `fielddata=true` on
[`your_field_name`] in order to load  fielddata in memory by uninverting the
inverted index. Note that this can however use significant memory.
```



## 1.2 fielddata开启前的思考

在启用字段数据之前，请考虑为什么将文本字段用于聚合，排序或在脚本中使用。 这样做通常没有任何意义。

在索引之前会分析文本字段，以便可以通过搜索new或york来找到类似New York的值。 当您可能想要一个名为New York的存储桶时，此字段上的字词汇总将返回一个新的存储桶和一个New York存储桶。

相反，您应该有一个用于全文搜索的文本字段，以及一个为聚合启用doc_values的未经分析的关键字字段，如下所示：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_field": { 		//1
        "type": "text",
        "fields": {
          "keyword": { 		//2
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

> //1:使用my_field字段进行搜索。
>
>
> //2:使用my_field.keyword字段进行聚合，排序或在脚本中使用。



## 1.3 在text字段上开启fielddata

您可以使用PUT映射API在现有文本字段上启用字段数据，如下所示：

```json
PUT my_index/_mapping
{
  "properties": {
    "my_field": { //1
      "type":     "text",
      "fielddata": true  
    }
  }
}
```

> //1:您为my_field指定的映射应包括该字段的现有映射以及fielddata参数。



**fielddata_frequency_filter**

字段数据过滤可用于减少加载到内存中的字词数量，从而减少内存使用量。 可以按频率过滤字词：

频率过滤器允许您仅加载文档频率介于最小值和最大值之间的术语，这些术语可以表示为绝对数（当数字大于1.0时）或百分比（例如0.01为1％且1.0为100） ％）。 频率是按segment计算的。 百分比基于具有该字段值的文档数，而不是segment所有文档数。

通过使用min_segment_size指定片段应包含的最小文档数，可以完全排除小片段：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "tag": {
        "type": "text",
        "fielddata": true,
        "fielddata_frequency_filter": {
          "min": 0.001,
          "max": 0.1,
          "min_segment_size": 500
        }
      }
    }
  }
}
```


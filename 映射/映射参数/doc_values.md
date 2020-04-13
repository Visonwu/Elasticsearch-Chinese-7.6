# doc_values

默认情况下，大多数字段都已编入索引，这使它们可搜索。 倒排索引允许查询在唯一的术语排序列表中查找搜索词，并从中立即访问包含该词的文档列表。

对脚本中的字段值进行排序，聚合和访问需要不同的数据访问模式。 除了查找术语和查找文档以外，我们还需要能够查找文档并查找其在字段中具有的术语。

Doc values是在文档索引时构建的磁盘上数据结构，这使这种数据访问模式成为可能。 它们存储与`_source`相同的值，但是以面向列的方式存储，这对于排序和聚合而言更为有效。 几乎所有字段类型都支持Doc values，但text和annotated_text字段除外。

默认情况下，所有支持doc值的字段均已启用它们。 如果您确定不需要对字段进行排序或汇总，也不需要通过脚本访问字段值，则可以禁用doc值以节省磁盘空间：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "status_code": {   //1
        "type":       "keyword"
      },
      "session_id": {    //2
        "type":       "keyword",
        "doc_values": false
      }
    }
  }
}
```

> //1:status_code字段默认情况下启用了doc_values。
>
>
> //2:session_id已禁用doc_values，但仍可以查询。
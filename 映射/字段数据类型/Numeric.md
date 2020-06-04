# Numeric 数据类型

支持以下数字类型：

| `long`         | 有符号的64位整数，最小值为-2的63幂次方，最大值为2的63幂次方-1。 |
| -------------- | ------------------------------------------------------------ |
| `integer`      | 一个带符号的32位整数，最小值为-2的31幂次方，最大值为2的31幂次方-1。 |
| `short`        | 一个带符号的16位整数，最小值为-32,768，最大值为32,767。      |
| `byte`         | 一个带符号的8位整数，最小值为-128，最大值为127。 |
| `double`       | 一个双精度64位IEEE 754浮点数，限制为有限值。 |
| `float`        | 单精度32位IEEE 754浮点数，限制为有限值。 |
| `half_float`   | 半精度16位IEEE 754浮点数，限制为有限值。 |
| `scaled_float` | 由“ long”支持的浮点数，由固定的“ double”缩放因子缩放。 |

以下是使用数字字段配置映射的示例：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "number_of_bytes": {
        "type": "integer"
      },
      "time_in_seconds": {
        "type": "float"
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
    }
  }
}
```

> double，float和half_float类型认为-0.0和+0.0是不同的值。 结果，在-0.0上执行词条查询将不匹配+0.0，反之亦然。 范围查询也是如此：如果上限是-0.0，则+0.0将不匹配；如果下限是+0.0，则-0.0将不匹配。





## 1.我应该使用哪种类型？

就整数类型（字节，短整数，整数和长整数）而言，应选择足以满足用例的最小类型。 这将有助于索引和搜索更加有效。 但是请注意，存储是根据存储的实际值进行优化的，因此，选择一种类型而不是另一种类型不会影响存储需求。

对于浮点类型，使用缩放因子将浮点数据存储为整数通常更为有效，这是scaled_float类型在后台执行的操作。例如，价格字段可以存储在scaled_float中，并且scale_factor为100。所有API都可以像将字段存储为double一样工作，但在Elasticsearch的帮助下，价格为美分，price * 100，这是一个整数。这对节省磁盘空间很有帮助，因为整数比浮点数更容易压缩。也可以使用scaled_float来交换磁盘空间的准确性。例如，假设您正在跟踪cpu利用率为0到1之间的数字。cpu利用率是12.7％还是13％通常并不重要，因此您可以使用scaled_float并将scale_factor设置为100以舍入cpu利用率到最接近的百分比以节省空间。

如果scaled_float不适合，则应在浮点类型中选择足以满足用例的最小类型：double，float和half_float。这是一张表格，将这些类型进行比较以帮助做出决定。

![twRCjI.png](https://s1.ax1x.com/2020/06/04/twRCjI.png)

> 提示：
>
> 映射数字标识符
> 并非所有数字数据都应映射为数字字段数据类型。 Elasticsearch为范围查询优化数字字段，例如整数或长整数。 但是，keyword字段更适合term和其他 [term-level](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/term-level-queries.html)的查询。
>
> 范围查询中很少使用标识符，例如ISBN或产品ID。 但是，通常使用术语级查询来检索它们。
>
> 在以下情况下，请考虑将数字标识符映射为keyword：
>
> 您不打算使用range查询来搜索标识符数据。
> 快速检索很重要。 在keyword字段上进行字词查询搜索通常比在Numeric 字段上进行字词搜索更快。
> 如果不确定要使用哪个数据，可以使用[multi-field](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/multi-fields.html)将数据映射为keyword和Numeric 数据类型。



## 2. 数字字段的参数
数值类型接受以下参数：

| [`coerce`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/coerce.html) | 尝试将字符串转换为数字，并截断整数部分。 接受“ true”（默认）和“ false”。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/doc-values.html) | 该字段是否应该以colloumn-stride的方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？ 接受“ true”（默认）或“ false”。 |
| [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/ignore-malformed.html) | 如果为true，则忽略格式错误的数字。 如果为“ false”（默认），则格式错误的数字将引发异常并拒绝整个文档。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html) | 该字段是否可以搜索？ 接受“ true”（默认）和“ false”。         |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/null-value.html) | 接受与字段相同的“类型”的数字值，以替换任何显式的“空”值。 默认为“ null”，这意味着该字段被视为丢失。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html) | 字段值是否应与[`_source`](https_www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-source-field.html)字段分开存储和检索。 接受“ true”或“ false”（默认）。 |
| [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-field-meta.html) | 有关该字段的元数据。                                         |



## 3.scaled_float的参数
scaled_float接受一个附加参数：

| `scaling_factor` | 编码值时使用的比例因子。 值将在索引时间乘以该因子，并四舍五入为最接近的long值。 例如，一个带有“ scaleing_factor”为“ 10”的“ scaled_float”将在内部将“ 2.34”存储为“ 23”，并且所有搜索时间操作（查询，汇总，排序）都将像文档的值为“`” 2.3`。 较高的“ scaling_factor”值可提高准确性，但也会增加空间需求。 此参数是必需的。 |
| ---------------- | ------------------------------------------------------------ |
|                  |                                                              |


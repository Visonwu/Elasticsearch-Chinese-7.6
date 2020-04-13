# date

JSON没有日期数据类型，因此Elasticsearch中的日期可以是：

- 包含格式化日期的字符串，例如 “ 2015-01-01”或“ 2015/01/01 12:10:30”。
- 一个long number，代表从纪元以来的毫秒数。
- 一个Integer，表示纪元以来的秒数。

在内部，日期会转换为UTC（如果指定了时区），并存储为一个整数，表示从纪元以来的毫秒数。

日期查询在内部转换为这种long表示形式的范围查询，并且聚合和存储字段的结果将转换为字符串，具体取决于与该字段关联的日期格式。

> 注意：日期将始终以字符串形式呈现，即使最初在JSON文档中提供的日期是long 也是如此。



日期格式可以自定义，但是如果未指定格式，则使用默认格式：

`“ strict_date_optional_time || epoch_millis”`
这意味着它将接受带有可选时间戳记的日期，这些时间戳记与`strict_date_optional_time`或mimiseconds（自纪元）支持的格式一致。

例如：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "date": {
        "type": "date"  //1
      }
    }
  }
}

PUT my_index/_doc/1
{ "date": "2015-01-01" }  //2

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30Z" }  //3

PUT my_index/_doc/3
{ "date": 1420070400001 }  //4

GET my_index/_search
{
  "sort": { "date": "asc"}  //5
}
```

> //1:日期字段使用默认格式。
>
>
> //2:本文档使用简单的日期。
>
>
> //3:该文件包括一个时间。
>
>
> //4:本文档自纪元以来使用毫秒。
>
>
> //5:请注意，返回的排序值都以纪元来的毫秒排序（以毫秒为单位）。



## 1.1 多日期格式

可以使用`||`分隔指定多种格式 作为分隔符。 依次尝试每种格式，直到找到匹配的格式。 第一种格式将用于将自纪元以来的毫秒值转换回字符串

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "date": {
        "type":   "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}
```



## 1.2 日期字段的参数

日期字段接受以下参数：

| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) | 映射字段级查询时间的增加。接受浮点数，默认为1.0。            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) | 该字段是否应该以column stride fashion方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？接受true（默认）或false。 |
| [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html) | 可以解析的日期格式。默认为strict_date_optional_time\|\| epoch_milis |
| `locale`                                                     | 自几个月以来，解析日期时使用的语言环境在所有语言中的名称和/或缩写都不相同。默认为 [`ROOT` locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT), |
| [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-malformed.html) | 如果为true，则会忽略格式错误的数字。如果为false（默认），则格式错误的数字将引发异常并拒绝整个文档。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-index.html) | 该字段是否可以搜索？接受true（默认）和false。                |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html) | 接受采用配置格式之一的日期值作为该字段，以替换任何显式的空值。默认为null，这意味着该字段被视为丢失。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) | 是否应与_source字段分开存储和检索字段值。接受true或false（默认）。 |
| [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-meta.html) | 按日期字段有关字段的元数据：                                 |
# Date nanoseconds

此数据类型是日期数据类型的补充。 但是，两者之间有重要区别。 现有的日期数据类型以毫秒为单位存储日期。 date_nanos数据类型以纳秒级分辨率存储日期，这限制了它的日期范围从大约1970到2262，因为日期仍存储为自纪元以来的较长的代表纳秒的时间。

纳秒级的查询在内部转换为对此long表示形式的范围查询，并且聚合和存储的字段的结果将转换为字符串，具体取决于与该字段关联的日期格式。

日期格式可以自定义，但是如果未指定格式，则使用默认格式：

`“ strict_date_optional_time || epoch_millis”`
这意味着它将接受带有可选时间戳记的日期，该时间戳记与strict_date_optional_time支持的格式一致，包括自小数点后最多九秒或毫秒（因此会损失纳秒精度）。

例如：

```json
PUT my_index?include_type_name=true
{
  "mappings": {
    "_doc": {
      "properties": {
        "date": {
          "type": "date_nanos"  //1
        }
      }
    }
  }
}

PUT my_index/_doc/1
{ "date": "2015-01-01" } //2

PUT my_index/_doc/2
{ "date": "2015-01-01T12:10:30.123456789Z" } //3

PUT my_index/_doc/3
{ "date": 1420070400 } //4

GET my_index/_search
{
  "sort": { "date": "asc"} //5
}

GET my_index/_search
{
  "script_fields" : {
    "my_field" : {
      "script" : {
        "lang" : "painless",
        "source" : "doc['date'].value.nano"  //6
      }
    }
  }
}

GET my_index/_search
{
  "docvalue_fields" : [
    {
      "field" : "my_ip_field",
      "format": "strict_date_time"  //7
    }
  ]
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
> //4:本文档自纪元以来的毫秒。
>
>
> //5:请注意，自纪元以来，返回的排序值都以纳秒为单位。
>
>
> //6:在脚本中访问日期的纳秒部分
>
>
> //7: 使用doc值字段，可将其设置为纳秒级分辨率

您还可以指定多个日期格式，以||分隔。 可以使用与日期字段相同的映射参数。

## 1.1 限制

即使使用date_nanos字段，聚合仍以毫秒为单位。
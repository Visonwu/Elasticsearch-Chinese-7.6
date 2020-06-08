# ignore_malformed

有时，您对收到的数据没有太多控制权。 一个用户可以发送一个作为日期的登录字段，另一个用户发送一个作为电子邮件地址的登录字段。

默认情况下，尝试将错误的数据类型索引到字段中会引发异常，并拒绝整个文档。 如果将ignore_malformed参数设置为true，则可以忽略该异常。 格式错误的字段未编制索引，但文档中的其他字段已正常处理。



例如：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "number_one": {
        "type": "integer",
        "ignore_malformed": true
      },
      "number_two": {
        "type": "integer"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text":       "Some text value",
  "number_one": "foo" 		//1
}

PUT my_index/_doc/2
{
  "text":       "Some text value",
  "number_two": "foo" 		//2
}
```

> //1: 该文档将为文本字段建立索引，但不会为number_one字段建立索引。
>
>
> //2: 该文件将被拒绝，因为number_two不允许格式错误的值。



以下映射类型当前支持ignore_malformed设置：

- **Numeric**:  long`, `integer`, `short`, `byte`, `double`, `float`, `half_float`, `scaled_float

- **Date**:  date

- **Date nanoseconds**:   date_nanos

- **Geo-point**: `geo_point` 纬度/经度点

- **Geo-shape**: `geo_shape`用于多边形等复杂形状

- **IP**：`ip`用于IPv4和IPv6地址



> 注：可以使用PUT映射API在现有字段上更新ignore_malformed设置值。



## 1.索引级默认
可以在索引级别上设置index.mapping.ignore_malformed设置，以全局忽略所有允许的映射类型上格式错误的内容。 如果设置为索引级别，则不支持该设置的映射类型将忽略它。

```json
PUT my_index
{
  "settings": {
    "index.mapping.ignore_malformed": true 	//1
  },
  "mappings": {
    "properties": {
      "number_one": { 			//1
        "type": "byte"
      },
      "number_two": {
        "type": "integer",
        "ignore_malformed": false 		//2
      }
    }
  }
}
```

> //1: number_one字段继承了索引级别的设置。
>
>
> //2: number_two字段将覆盖索引级别的设置以关闭ignore_malformed。



## 2.处理格式错误的字段
打开ignore_malformed时，在索引编制时将静默忽略格式错误的字段。 只要有可能，建议保留包含格式错误字段的文档数，否则对该字段的查询将变得毫无意义。 通过使用特殊_ignored字段中的 `exists`,`term` or `terms`，Elasticsearch可以轻松地检查有多少文档的字段格式错误。





## 3. JSON对象的限制
您不能对以下数据类型使用ignore_malformed：

- [Nested datatype](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/nested.html)
- [Object datatype](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/object.html)
- [Range datatypes](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/range.html)



您也不能使用ignore_malformed忽略提交到错误数据类型字段的JSON对象。 JSON对象是用大括号“ {}”括起来的任何数据，并且包括映射到嵌套，对象和范围数据类型的数据。

如果将JSON对象提交到不支持的字段，则无论ignore_malformed设置如何，Elasticsearch都会返回错误并拒绝整个文档。


























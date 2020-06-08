# null_value

空值不能被索引或搜索。 当字段设置为null（或空数组或null值的数组）时，将其视为该字段没有值。

使用null_value参数可以用指定的值替换显式的空值，以便可以对其进行索引和搜索。 例如：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "status_code": {
        "type":       "keyword",
        "null_value": "NULL" 			//1
      }
    }
  }
}

PUT my_index/_doc/1
{
  "status_code": null
}

PUT my_index/_doc/2
{
  "status_code": [] 				//2
}

GET my_index/_search
{
  "query": {
    "term": {
      "status_code": "NULL" 		//3
    }
  }
}
```

> //1: 用术语NULL替换显式的null值。
>
>
> //2: 空数组不包含显式的null，因此不会被null_value取代。
>
>
> //3: 查询NULL将返回文档1，但不会返回文档2。



null_value必须与字段具有相同的数据类型。 例如，长字段不能包含字符串null_value。



> null_value仅影响索引数据的方式，不会修改_source文档。








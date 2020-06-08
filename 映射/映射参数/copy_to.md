# copy_to

使用copy_to参数可以将多个字段的值复制到组字段中，然后可以将其作为单个字段进行查询。 例如，可以将first_name和last_name字段复制到full_name字段，如下所示：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text",
        "copy_to": "full_name" 		//1
      },
      "last_name": {
        "type": "text",
        "copy_to": "full_name" 		//1
      },
      "full_name": {
        "type": "text"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "first_name": "John",
  "last_name": "Smith"
}

GET my_index/_search
{
  "query": {
    "match": {
      "full_name": { 				//2
        "query": "John Smith",
        "operator": "and"
      }
    }
  }
}
```

> //1: 将first_name和last_name字段的值复制到full_name字段。
>
> //2: 仍然可以分别查询first_name和last_name字段的名字和姓氏，但是可以查询full_name字段的名字和姓氏。



一些要点：

- 复制的是字段值，而不是术语（由分析过程得出）。
- 原始的_source字段将不会修改以显示复制的值。
- 可以使用“ copy_to”将相同的值复制到多个字段：[“ field_1”，“ field_2”]
- 您不能通过中间字段（例如，在field_1上的copy_to到field_2以及在field_2上的copy_to到field_3）进行递归复制，因为期望到field_1的索引将在field_3中完成，而是直接使用copy_to从原始字段复制到多个字段。








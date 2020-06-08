# enabled

Elasticsearch尝试为您提供的所有字段建立索引，但有时您只想存储该字段而不建立索引。 例如，假设您将Elasticsearch用作Web会话存储。 您可能希望为会话ID和上次更新时间建立索引，但是不需要查询或运行会话数据本身的聚合。

启用的设置仅可应用于顶级映射定义和[`object`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/object.html) 字段，导致Elasticsearch完全跳过对字段内容的解析。 仍然可以从 [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-source-field.html)字段中检索JSON，但是无法搜索或以其他任何方式存储JSON：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "user_id": {
        "type":  "keyword"
      },
      "last_updated": {
        "type": "date"
      },
      "session_data": { 			//1
        "type": "object",
        "enabled": false
      }
    }
  }
}

PUT my_index/_doc/session_1
{
  "user_id": "kimchy",
  "session_data": { 				//2
    "arbitrary_object": {
      "some_array": [ "foo", "bar", { "baz": 2 } ]
    }
  },
  "last_updated": "2015-12-06T18:20:22"
}

PUT my_index/_doc/session_2
{
  "user_id": "jpountz",
  "session_data": "none", 				//3
  "last_updated": "2015-12-06T18:22:13"
}
```

> //1: session_data字段被禁用。
>
>
> //2: 任何任意数据都可以传递到session_data字段，因为它将被完全忽略。
>
>
> //3: session_data也将忽略不是JSON对象的值。	





整个映射也可能被禁用，在这种情况下，文档存储在_source字段中，这意味着可以对其进行检索，但是其任何内容都不会以任何方式建立索引：

```json
UT my_index
{
  "mappings": {
    "enabled": false 					//1
  }
}

PUT my_index/_doc/session_1
{
  "user_id": "kimchy",
  "session_data": {
    "arbitrary_object": {
      "some_array": [ "foo", "bar", { "baz": 2 } ]
    }
  },
  "last_updated": "2015-12-06T18:20:22"
}

GET my_index/_doc/session_1 				//2
		
GET my_index/_mapping 						//3
```

> //1: 整个映射被禁用。
>
>
> //2: 可以检索该文档。
>
>
> //3: 检查映射显示没有添加任何字段。



现有字段的已启用设置和顶级映射定义无法更新。

请注意，由于Elasticsearch完全跳过了对字段内容的解析，因此可以将非对象数据添加到禁用的字段中：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "session_data": {
        "type": "object",
        "enabled": false
      }
    }
  }
}

PUT my_index/_doc/session_1
{
  "session_data": "foo bar" 			//1
}
```

> //1: 即使session_data包含非对象数据，该文档也已成功添加。








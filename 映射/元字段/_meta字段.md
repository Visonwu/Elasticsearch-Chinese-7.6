# _meta 字段

映射类型可以具有与之关联的自定义元数据。 Elasticsearch完全不使用这些，但是可用于存储特定于应用程序的元数据，例如文档所属的类：

```json
PUT my_index
{
  "mappings": {
    "_meta": { 				//1
      "class": "MyApp::User",
      "version": {
        "min": "1.0",
        "max": "1.3"
      }
    }
  }
}
```

> 可以使用 [GET mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-get-mapping.html) API检索此_meta信息。

可以使用 [PUT mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-put-mapping.html) API在现有类型上更新_meta字段：

```json
PUT my_index/_mapping
{
  "_meta": {
    "class": "MyApp2::User3",
    "version": {
      "min": "1.3",
      "max": "1.5"
    }
  }
}
```


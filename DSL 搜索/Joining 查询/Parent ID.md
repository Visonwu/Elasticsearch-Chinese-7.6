# 父ID 查询

返回加入特定父文档的子文档。 您可以使用联接字段映射在同一索引中的文档之间创建父子关系。



## 1.1 请求案例

**索引创建**

要使用parent_id查询，您的索引必须包含一个联接字段映射。 若要查看如何为parent_id查询设置索引，请尝试以下示例

1.使用联接字段映射创建索引。

```json
PUT /my-index
{
    "mappings": {
        "properties" : {
            "my-join-field" : {
                "type" : "join",
                "relations": {
                    "my-parent": "my-child"
                }
            }
        }
    }
}
```



2.索引ID为1的父文档。

```json
PUT /my-index/_doc/1?refresh
{
  "text": "This is a parent document.",
  "my-join-field": "my-parent"
}
```



3.索引父文档的子文档。

```json
PUT /my-index/_doc/2?routing=1&refresh
{
  "text": "This is a child document.",
  "my_join_field": {
    "name": "my-child",
    "parent": "1"
  }
}
```



## 1.2 查询案例

以下搜索返回ID为1的父文档的子文档。

```json
GET /my-index/_search
{
  "query": {
      "parent_id": {
          "type": "my-child",
          "id": "1"
      }
  }
}
```



## 1.3  parent ID的高级参数

**`type`**

（必需，字符串）为[join](https://www.elastic.co/guide/zh-CN/elasticsearch/reference/7.6/parent-join.html)字段映射的子关系的名称。

**`id`**

（必需，字符串）父文档的ID。 查询将返回此父文档的子文档。

**`ignore_unmapped`**

（可选，布尔值）指示是否忽略未映射的“类型”而不返回任何文档而不是错误。 默认为`false`。

  如果为false，则在未映射type的情况下，Elasticsearch返回错误。

  您可以使用此参数查询可能不包含“ type”的多个索引。
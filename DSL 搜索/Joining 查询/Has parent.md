# Has parent 查询

返回其子级父文档与提供的查询匹配的子文档。 您可以使用联接字段映射在同一索引中的文档之间创建父子关系。

因为它执行联接，所以has_parent查询比其他查询慢。 随着匹配父文档数量的增加，其性能会下降。 搜索中的每个has_parent查询都会大大增加查询时间。



## 1.1 搜索案例

**创建索引**

要使用has_parent查询，您的索引必须包含一个联接字段映射。 例如：

```json
PUT /my-index
{
    "mappings": {
        "properties" : {
            "my-join-field" : {
                "type" : "join",
                "relations": {
                    "parent": "child"
                }
            },
            "tag" : {
                "type" : "keyword"
            }
        }
    }
}
```



**查询例子**

```json
GET /my-index/_search
{
    "query": {
        "has_parent" : {
            "parent_type" : "parent",
            "query" : {
                "term" : {
                    "tag" : {
                        "value" : "Elasticsearch"
                    }
                }
            }
        }
    }
}
```



## 1.2 has_parent的高级参数



**`parent_type`**

（必需，string）为join字段映射的父级关系的名称。

**`query`**

（必需,查询对象）查询您希望在“ parent_type”字段的父文档上运行的查询。 如果父文档与搜索匹配，则查询返回其子文档。

**`score`**

(可选，布尔值）指示是否将匹配的父文档的[相关性得分](<https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-filter-context.html>)汇总到其子文档中。 默认为`false`。

如果为false，Elasticsearch将忽略父文档的相关性得分。 Elasticsearch还会为每个子文档分配一个相关性得分，该相关性得分等于“查询”的“提升”，默认为1。

如果为true，则将匹配的父文档的相关性分数汇总到其子文档的相关性分数中。

**`ignore_unmapped`**

（可选，布尔值）指示是否忽略未映射的“ parent_type”并且不返回任何文档而不是返回错误。 默认为`false`。

如果为“ false”，则如果未映射“ parent_type”，Elasticsearch将返回错误。

您可以使用此参数查询可能不包含“ parent_type”的多个索引。



## 1.3 Notes

### 1) sorting

您不能使用[标准排序选项](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-request-body.html#request-body-search-sort)对has_parent查询的结果进行排序。

如果您需要按其父文档中的字段对返回的文档进行排序，请使用`function_score`查询并按`_score`进行排序。 例如，以下查询按其父文档的`view_count`字段对返回的文档进行排序。

```json
GET /_search
{
    "query": {
        "has_parent" : {
            "parent_type" : "parent",
            "score" : true,
            "query" : {
                "function_score" : {
                    "script_score": {
                        "script": "_score * doc['view_count'].value"
                    }
                }
            }
        }
    }
}
```


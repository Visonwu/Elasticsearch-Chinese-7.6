# 嵌套查询

包装另一个查询以搜索[嵌套](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/nested.html))字段。

嵌套查询搜索嵌套的字段对象，就像它们被索引为单独的文档一样。 如果对象与搜索匹配，则嵌套查询返回根父文档。



## 1.1 请求示例

**创建索引**

要使用嵌套查询，您的索引必须包含一个嵌套字段映射。 例如

```json
PUT /my_index
{
    "mappings" : {
        "properties" : {
            "obj1" : {
                "type" : "nested"
            }
        }
    }
}
```

## 1.2 查询示例

```json
GET /my_index/_search
{
    "query":  {
        "nested" : {
            "path" : "obj1",
            "query" : {
                "bool" : {
                    "must" : [
                    { "match" : {"obj1.name" : "blue"} },
                    { "range" : {"obj1.count" : {"gt" : 5}} }
                    ]
                }
            },
            "score_mode" : "avg"
        }
    }
}
```



## 1.3 nested的 高级参数

**`path`**

（必需，字符串）要搜索的嵌套对象的路径。

**`query`**

（必需，查询对象）查询您希望在“path”中的嵌套对象上运行的查询。 如果对象与搜索匹配，则“嵌套”查询返回根父文档。

您可以使用包含完整路径的点符号来搜索嵌套字段，例如`obj1.name`。

自动支持并检测到多级嵌套，如果一个嵌套查询存在于另一个嵌套查询中，则内部嵌套查询会自动匹配相关的嵌套级别，而不是根。

请参阅[多层嵌套查询](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-nested-query.html#multi-level-nested-query-ex) 一个例子。

**`score_mode`**

（可选，字符串）指示匹配子对象的分数如何影响根父文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-filter-context.html)。 有效值为：

- **`avg` (Default)**

  使用所有匹配的子对象的平均相关性得分。

- **`max`**

  使用所有匹配的子对象中最高的相关性得分。

- **`min`**

  使用所有匹配的子对象中最低的相关性得分。

- **`none`**

  不要使用匹配的子对象的相关性分数。 该查询为父文档分配得分“ 0”。

- **`sum`**

     将所有匹配的子对象的相关性得分相加

**`ignore_unmapped`**

（可选，布尔值）指示是否忽略未映射的“路径”而不返回任何文档而不是错误。 默认为`false`。

  如果为false，则当path为未映射字段时，Elasticsearch返回错误。

  您可以使用此参数查询可能不包含“ path”字段的多个索引。



## 1.4 Notes

### 1)多级嵌套查询

要查看多级嵌套查询的工作方式，首先需要一个具有嵌套字段的索引。 以下请求使用嵌套的make和model字段定义drivers索引的映射。

```json
PUT /drivers
{
    "mappings" : {
        "properties" : {
            "driver" : {
                "type" : "nested",
                "properties" : {
                    "last_name" : {
                        "type" : "text"
                    },
                    "vehicle" : {
                        "type" : "nested",
                        "properties" : {
                            "make" : {
                                "type" : "text"
                            },
                            "model" : {
                                "type" : "text"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

接下来，将一些文档编入drivers索引中。

```json
PUT /drivers/_doc/1
{
  "driver" : {
        "last_name" : "McQueen",
        "vehicle" : [
            {
                "make" : "Powell Motors",
                "model" : "Canyonero"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}

PUT /drivers/_doc/2?refresh
{
  "driver" : {
        "last_name" : "Hudson",
        "vehicle" : [
            {
                "make" : "Mifune",
                "model" : "Mach Five"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}
```

现在，您可以使用多级嵌套查询来根据make和model字段来匹配文档。

```json
GET /drivers/_search
{
    "query" : {
        "nested" : {
            "path" : "driver",
            "query" : {
                "nested" : {
                    "path" :  "driver.vehicle",
                    "query" :  {
                        "bool" : {
                            "must" : [
                                { "match" : { "driver.vehicle.make" : "Powell Motors" } },
                                { "match" : { "driver.vehicle.model" : "Canyonero" } }
                            ]
                        }
                    }
                }
            }
        }
    }
}
```

搜索请求返回以下响应：

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 3.7349272,
    "hits" : [
      {
        "_index" : "drivers",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 3.7349272,
        "_source" : {
          "driver" : {
            "last_name" : "McQueen",
            "vehicle" : [
              {
                "make" : "Powell Motors",
                "model" : "Canyonero"
              },
              {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
              }
            ]
          }
        }
      }
    ]
  }
}
```


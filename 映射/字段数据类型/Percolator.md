#  Percolator类型

percolator字段类型将json结构解析为native查询并存储该查询，以便[percolate query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-percolate-query.html)可以使用它来匹配提供的文档。

包含json对象的任何字段都可以配置为percolator字段。 percolator字段类型没有设置。 仅配置percolator字段类型就足以指示Elasticsearch将字段视为查询。

如果以下映射为query字段配置了percolator字段类型：

```json
PUT my_index
{
    "mappings": {
        "properties": {
            "query": {
                "type": "percolator"
            },
            "field": {
                "type": "text"
            }
        }
    }
}
```

然后，您可以为查询建立索引：

```json
PUT my_index/_doc/match_value
{
    "query" : {
        "match" : {
            "field" : "value"
        }
    }
}
```

渗滤器查询中引用的字段必须已经存在于与用于渗滤的索引关联的映射中。 为了确保这些字段存在，请通过[create index](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-create-index.html) or [put mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/indices-put-mapping.html) APIs添加或更新映射。

## 1.重新索引您的percolator查询

​	有时需要重新索引percolator查询才能从新版本中对percolator字段类型进行的改进中受益。

可以使用 [reindex api](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-reindex.html).重新索引percolator查询。 让我们看一下带有percolator字段类型的以下索引：

```json
PUT index
{
  "mappings": {
    "properties": {
      "query" : {
        "type" : "percolator"
      },
      "body" : {
        "type": "text"
      }
    }
  }
}

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index",
        "alias": "queries" 
      }
    }
  ]
}

PUT queries/_doc/1?refresh
{
  "query" : {
    "match" : {
      "body" : "quick brown fox"
    }
  }
}
```

> 始终建议为索引定义别名，这样在进行重新索引的情况下，无需更改系统/应用程序即可知道percolator 查询现在位于不同的索引中。



假设您要升级到新的主要版本，并且为了使新的Elasticsearch版本仍然能够读取您的查询，您需要将查询重新索引为当前Elasticsearch版本的新索引：

```json
PUT new_index
{
  "mappings": {
    "properties": {
      "query" : {
        "type" : "percolator"
      },
      "body" : {
        "type": "text"
      }
    }
  }
}

POST /_reindex?refresh
{
  "source": {
    "index": "index"
  },
  "dest": {
    "index": "new_index"
  }
}

POST _aliases
{
  "actions": [  //1
    {
      "remove": {
        "index" : "index",
        "alias": "queries"
      }
    },
    {
      "add": {
        "index": "new_index",
        "alias": "queries"
      }
    }
  ]
}
```

> //1: 如果您有别名，请不要忘记将其指向新索引。

通过查询alias 执行 percolate 查询：

```json
GET /queries/_search
{
  "query": {
    "percolate" : {
      "field" : "query",
      "document" : {
        "body" : "fox jumps over the lazy dog"
      }
    }
  }
}
```

现在从新索引返回匹配项：

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.13076457,
    "hits": [
      {
        "_index": "new_index",  //1
        "_type": "_doc",
        "_id": "1",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "body": "quick brown fox"
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [0]
        }
      }
    ]
  }
}
```

> //1:在，从新索引中显示了percolator查询命中率。


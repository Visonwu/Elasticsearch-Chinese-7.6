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



## 2. 优化查询时间文本分析
​		当percolator验证percolator候选匹配时，它将进行解析，执行查询时间文本分析并实际上对要渗滤的文档运行percolator查询。 对于每个候选匹配项以及每次执行渗滤查询时，都会执行此操作。 如果您的查询时间文本分析是查询解析中相对昂贵的一部分，那么在进行渗滤时，文本分析可能成为花费时间的主要因素。 当percolator最终验证许多候选percolator查询匹配时，此查询解析开销可能变得很明显。

​		避免在过滤时间进行文本分析中最昂贵的部分。在为渗滤器查询建立索引时，可以选择执行文本分析的昂贵部分。这需要使用两个不同的分析仪。第一个分析器实际上执行需要执行的文本分析（昂贵的部分）。第二个分析器（通常为空白）仅拆分第一个分析器已生成的生成令牌。然后，在为percolator查询建立索引之前，应使用分析api和更昂贵的分析器来分析查询文本。分析api的结果（令牌）应用于替代percolator查询中的原始查询文本。重要的是，现在应将查询配置为从映射中覆盖分析器，并仅覆盖第二个分析器。大多数基于文本的查询都支持分析器选项（match，query_string，simple_query_string）。使用这种方法，昂贵的文本分析将执行一次而不是多次。

让我们通过一个简化的示例演示此工作流程。

假设我们要为以下percolator 查询建立索引：

```json
{
  "query" : {
    "match" : {
      "body" : {
        "query" : "missing bicycles"
      }
    }
  }
}
```

这些设置和映射：

```json
PUT /test_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer" : {
          "tokenizer": "standard",
          "filter" : ["lowercase", "porter_stem"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "query" : {
        "type": "percolator"
      },
      "body" : {
        "type": "text",
        "analyzer": "my_analyzer"   //1
      }
    }
  }
}
```

> //1: 出于本示例的目的，该分析仪被认为是昂贵的。

首先，我们需要在索引之前使用分析API来执行文本分析：

```json
POST /test_index/_analyze
{
  "analyzer" : "my_analyzer",
  "text" : "missing bicycles"
}
```

结果如下：

```json
{
  "tokens": [
    {
      "token": "miss",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "bicycl",
      "start_offset": 8,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

返回顺序中的所有令牌都需要替换percolator查询中的查询文本：

```json
PUT /test_index/_doc/1?refresh
{
  "query" : {
    "match" : {
      "body" : {
        "query" : "miss bicycl",
        "analyzer" : "whitespace"   //1
      }
    }
  }
}
```

> //1: 在此处选择一个空白分析器很重要，否则将使用映射中定义的分析器，这会打乱使用此工作流程的目的。 请注意，空格是内置的分析器，如果需要使用其他分析器，则需要先在索引设置中对其进行配置。



在为percolator流程建立索引之前，应对每个percolator查询进行分析api。

在渗滤时间没有任何变化，并且`percolate` 查询可以正常定义：

```json
GET /test_index/_search
{
  "query": {
    "percolate" : {
      "field" : "query",
      "document" : {
        "body" : "Bycicles are missing"
      }
    }
  }
}
```

结果是这样的：

```json
{
  "took": 6,
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
        "_index": "test_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.13076457,
        "_source": {
          "query": {
            "match": {
              "body": {
                "query": "miss bicycl",
                "analyzer": "whitespace"
              }
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

## 3.优化通配符查询
对于通配符，通配符查询比其他查询昂贵，尤其是在通配符表达式较大的情况下。

对于带有前缀`wildcard` 表达式的通配符查询或仅带`prefix` 查询的情况，可以使用edge_ngram令牌过滤器在配置了edge_ngram令牌过滤器的字段上用常规术语查询替换这些查询。

使用自定义分析设置创建索引：

```json
UT my_queries1
{
  "settings": {
    "analysis": {
      "analyzer": {
        "wildcard_prefix": { 	//1
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "wildcard_edge_ngram"
          ]
        }
      },
      "filter": {
        "wildcard_edge_ngram": { 		//2
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 32
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "query": {
        "type": "percolator"
      },
      "my_field": {
        "type": "text",
        "fields": {
          "prefix": { 	//3
            "type": "text",
            "analyzer": "wildcard_prefix",
            "search_analyzer": "standard"
          }
        }
      }
    }
  }
}
```

> //1: 生成仅在索引时间使用的前缀标记的分析器。
>
>
> //2: 根据前缀搜索需要增加min_gram并减少max_gram设置。
>
>
> //3: 此多字段应用于通过术语或匹配查询而不是前缀或通配符查询来进行前缀搜索。

然后，代替索引以下查询：

```json
{
  "query": {
    "wildcard": {
      "my_field": "abc*"
    }
  }
}
```

下面的该查询应建立索引：

```json
PUT /my_queries1/_doc/1?refresh
{
  "query": {
    "term": {
      "my_field.prefix": "abc"
    }
  }
}
```

这种方式可以比第一个查询更有效地处理第二个查询。

以下搜索请求将与先前索引的过滤器查询匹配：

```json
GET /my_queries1/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "my_field": "abcd"
      }
    }
  }
}
```

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.18864399,
    "hits": [
      {
        "_index": "my_queries1",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.18864399,
        "_source": {
          "query": {
            "term": {
              "my_field.prefix": "abc"
            }
          }
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        }
      }
    ]
  }
}
```

也可以使用相同的技术来加速后缀通配符搜索。 通过在edge_ngram令牌过滤器之前使用反向令牌过滤器。

```json
PUT my_queries2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "wildcard_suffix": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "reverse",
            "wildcard_edge_ngram"
          ]
        },
        "wildcard_suffix_search_time": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "reverse"
          ]
        }
      },
      "filter": {
        "wildcard_edge_ngram": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 32
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "query": {
        "type": "percolator"
      },
      "my_field": {
        "type": "text",
        "fields": {
          "suffix": {
            "type": "text",
            "analyzer": "wildcard_suffix",
            "search_analyzer": "wildcard_suffix_search_time"   //1
          }
        }
      }
    }
  }
}
```

> //1: 在搜索时也需要自定义分析器，因为否则查询条件将不会被反转，否则将与保留的后缀标记不匹配。

然后，代替索引以下查询：

```
{
  "query": {
    "wildcard": {
      "my_field": "*xyz"
    }
  }
}
```

下面的查询应建立索引：

```json
PUT /my_queries2/_doc/2?refresh
{
  "query": {
    "match": { 			//1
      "my_field.suffix": "xyz"
    }
  }
}
```

> //1: 应该使用匹配查询代替术语查询，因为文本分析需要反转查询术语。

以下搜索请求将与先前索引的过滤器查询匹配：

```json
GET /my_queries2/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "my_field": "wxyz"
      }
    }
  }
}
```



## 4.专用Percolator指数
渗滤查询可以添加到任何索引。 除了将渗滤查询添加到数据所在的索引中之外，还可以将这些查询添加到专用索引中。 这样做的好处是，该专用Percolator 索引可以具有自己的索引设置（例如，主分片和副本分片的数量）。 如果选择使用专用的Percolator 索引，则需要确保正常索引的映射在Percolator 索引上也可用。 否则，渗滤查询可能会被错误地解析。



## 5.强制将未映射的字段处理为字符串
在某些情况下，不知道将注册哪种类型的Percolator 查询，并且如果Percolator 查询所引用的字段不存在任何字段映射，则添加Percolator 查询将失败。 这意味着需要更新映射以使字段具有适当的设置，然后可以添加Percolator 查询。 但是有时将所有未映射的字段都当作默认文本字段来处理就足够了。 在这种情况下，可以将index.percolator.map_unmapped_fields_as_text设置配置为true（默认为false），然后如果在percolator查询中引用的字段不存在，则将其作为默认文本字段进行处理，以便添加percolator查询不会失败。



## 6.局限性
###  6.1 父母/子女
​		由于percolate查询一次只处理一个文档，因此它不支持针对子文档（例如has_child和has_parent）运行的查询和过滤器。

### 6.2 获取查询
​		有许多查询在查询解析期间通过get调用获取数据。例如，在使用术语查找时使用术语查询，在使用索引脚本时使用模板查询，在使用预索引形状时使用geo_shape。当这些查询由percolator字段类型索引时，get调用将执行一次。因此，每次percolator查询评估这些查询时，都将使用索引时的提取条件，形状等。需要注意的重要一点是，每次在主分片和副本分片上都为percolator查询建立索引时，都会获取这些查询所做的术语，因此，如果在建立索引时更改了源索引，则在分片副本之间实际被索引的术语可能会有所不同。

### 6.3 脚本查询
​	脚本查询中的脚本只能访问doc值字段。 percolate查询将提供的文档索引为内存索引。此内存中索引不支持存储的字段，因此，不存储_source字段和其他存储的字段。这就是为什么在脚本查询中_source和其他存储的字段不可用的原因。

### 6.4 字段别名

​		包含字段别名的percolator查询可能无法始终按预期方式运行。特别是，如果注册了一个包含字段别名的percolator查询，然后在映射中更新该别名以引用其他字段，则存储的查询仍将引用原始目标字段。要获取对字段别名的更改，必须显式重新索引percolator查询


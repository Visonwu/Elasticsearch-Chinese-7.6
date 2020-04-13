# join 数据类型

join数据类型是一个特殊字段，可在相同索引的文档中创建父/子关系。 关系部分在文档中定义了一组可能的关系，每个关系都是父名称和子名称。 父/子关系可以定义如下：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_id": {
        "type": "keyword"
      },
      "my_join_field": {  //1
        "type": "join",
        "relations": {
          "question": "answer"  //2 
        }
      }
    }
  }
}
```

> //1:字段的名称
>
> //2.:定义单个关系，其中question是answer的父级。



要使用连接为文档编制索引，必须在source中提供关系的name和文档的可选父级。 例如，以下示例在question上下文中创建两个父文档：

```json
PUT my_index/_doc/1?refresh
{
  "my_id": "1",
  "text": "This is a question",
  "my_join_field": {
    "name": "question"  //1
  }
}

PUT my_index/_doc/2?refresh
{
  "my_id": "2",
  "text": "This is another question",
  "my_join_field": {
    "name": "question"
  }
}
```

> //1: 这个文档是question文档

为父级文档建立索引时，可以选择仅指定关系的名称作为快捷方式，而不必将其封装在普通的对象符号中：

```json
PUT my_index/_doc/1?refresh
{
  "my_id": "1",
  "text": "This is a question",
  "my_join_field": "question"  //1
}

PUT my_index/_doc/2?refresh
{
  "my_id": "2",
  "text": "This is another question",
  "my_join_field": "question"
}
```

> //1.父文档的简单表示法仅使用关系名称。



为子级建立索引时，必须在_source中添加关系的名称以及文档的父级ID。

> 注：需要在同一分片中索引父项的谱系，因此您必须始终使用其更大的父项ID路由子文档。



例如，以下示例显示了如何索引两个子文档：

```json
PUT my_index/_doc/3?routing=1&refresh  //1
{
  "my_id": "3",
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer",  //2
    "parent": "1" 		//3
  }
}

PUT my_index/_doc/4?routing=1&refresh
{
  "my_id": "4",
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}
```

> //1:路由值是强制性的，因为父文档和子文档必须在同一分片上建立索引
>
>
> //2:question是此文档的联接名称
>
>
> //3:该子文档的父ID



## 1.1 父级-join 性能

得良好性能的关键是将数据反规范化为文档。 每个联接字段，has_child或has_parent查询都会对您的查询性能造成重大影响。

联接字段有意义的唯一情况是，如果您的数据包含一对多关系，其中一个实体明显多于另一个实体。 这种情况的一个例子是产品的用例以及这些产品的报价。 如果报价远远超过产品数量，则有必要将产品建模为父文档，将报价建模为子文档。



## 1.2 父级-join 限制

- 每个索引仅允许一个连接字段映射。
- 父文档和子文档必须在同一分片上建立索引。 这意味着在获取，删除或更新子文档时需要提供相同的路由值。
- 一个元素可以有多个子级，但只能有一个父级。
- 可以向现有联接字段添加新关系。
- 也可以将子元素添加到现有元素中，但前提是该元素已经是父元素。



## 1.3 通过父级-join 搜索

父联接创建一个字段以索引文档中关系的名称（my_parent，my_child等）。

它还为每个父/子关系创建一个字段。 该字段的名称是连接字段的名称，后跟＃和关系中的父级名称。 因此，例如对于my_parent→[my_child，another_child]关系，联接字段将创建一个名为my_join_field＃my_parent的附加字段。

如果文档是子文档（my_child或another_child），则此字段包含文档链接到的父_id；如果文档是父文档（my_parent），则包含文档的_id。

搜索包含联接字段的索引时，始终在搜索响应中返回以下两个字段：

```json
GET my_index/_search
{
  "query": {
    "match_all": {}
  },
  "sort": ["my_id"]
}
```

会返回：

```
{
    ...,
    "hits": {
        "total" : {
            "value": 4,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "my_index",
                "_type": "_doc",
                "_id": "1",
                "_score": null,
                "_source": {
                    "my_id": "1",
                    "text": "This is a question",
                    "my_join_field": "question"  //1
                },
                "sort": [
                    "1"
                ]
            },
            {
                "_index": "my_index",
                "_type": "_doc",
                "_id": "2",
                "_score": null,
                "_source": {
                    "my_id": "2",
                    "text": "This is another question",
                    "my_join_field": "question"  //2
                },
                "sort": [
                    "2"
                ]
            },
            {
                "_index": "my_index",
                "_type": "_doc",
                "_id": "3",
                "_score": null,
                "_routing": "1",
                "_source": {
                    "my_id": "3",
                    "text": "This is an answer",
                    "my_join_field": {
                        "name": "answer",  //3
                        "parent": "1"  		//4
                    }
                },
                "sort": [
                    "3"
                ]
            },
            {
                "_index": "my_index",
                "_type": "_doc",
                "_id": "4",
                "_score": null,
                "_routing": "1",
                "_source": {
                    "my_id": "4",
                    "text": "This is another answer",
                    "my_join_field": {
                        "name": "answer",
                        "parent": "1"
                    }
                },
                "sort": [
                    "4"
                ]
            }
        ]
    }
}
```

> //1: 该文件属于question联接
>
>
> //2:该文件属于question联接
>
>
> //3:该文档属于answer连接
>
>
> //4:子文档的链接父ID



## 1.4 父级-join查询和聚合

有关更多信息，请参见[has_child](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-has-child-query.html) 和[has_parent](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-has-parent-query.html)查询，[children](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-bucket-children-aggregation.html)聚合和[inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-request-body.html#parent-child-inner-hits) 匹配。

join字段的值可以在聚合和脚本中访问，并且可以通过[parent_id query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-parent-id-query.html)来查询：

```json
GET my_index/_search
{
  "query": {
    "parent_id": {  //1
      "type": "answer",
      "id": "1"
    }
  },
  "aggs": {
    "parents": {
      "terms": {
        "field": "my_join_field#question", //2
        "size": 10
      }
    }
  },
  "script_fields": {
    "parent": {
      "script": {
         "source": "doc['my_join_field#question']" //3
      }
    }
  }
}
```

> //1.查询父ID字段（另请参见[has_child](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-has-child-query.html)查询 和[has_parent](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-has-parent-query.html)查询）
>
>
> //2.在父ID字段上聚合（另请参阅 [`children`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-bucket-children-aggregation.html)聚合）
>
>
> //3.访问脚本中的父ID字段



## 1.5 Global ordinals

联接字段使用[global ordinals](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/eager-global-ordinals.html)来加快联接速度。 对分片进行任何更改后，都需要重建global ordinals。 分片中存储的父ID值越多，为联接字段重建global ordinals所需的时间就越长。

默认情况下，global ordinals会急于构建：如果索引已更改，则将在刷新过程中重建连接字段的global ordinals。 这会为刷新增加大量时间。 但是，大多数情况下这是正确的权衡，否则，当使用第一个父级联接查询或聚合时，将重建global ordinals。 这可能会给您的用户带来严重的延迟峰值，通常情况更糟，因为在发生多次写入时，可能会在单个刷新间隔内尝试重建连接字段的多个global ordinals。



如果不经常使用join字段并且频繁发生写入，则禁用积极加载可能是有道理的：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
           "question": "answer"
        },
        "eager_global_ordinals": false
      }
    }
  }
}
```

可以按每个父关系检查global ordinals使用的堆数量，如下所示：

```groovy
# Per-index
GET _stats/fielddata?human&fields=my_join_field#question

# Per-node per-index
GET _nodes/stats/indices/fielddata?human&fields=my_join_field#question
```



## 1.6 单父多子

也可以为单亲定义多个孩子：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": ["answer", "comment"]  //1
        }
      }
    }
  }
}
```

> `question` 是`answer` 和 `comment`的父级.



## 1.7 多个级别的父级联接

不建议使用多个级别的关系来复制关系模型。 每个关系级别都会在查询时增加内存和计算方面的开销。 如果您关心性能，则应该对数据进行非规范化。

多个级别的父/子

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": ["answer", "comment"],  //1
          "answer": "vote" //2
        }
      }
    }
  }
}
```

> //1:question 是answer和commit的父级
>
> //2：answer是vote的父级

上面表达的映射关系如下：

```
  question
    /    \
   /      \
comment  answer
           |
           |
          vote
```

索引孙文档需要一个等于祖父母（世系中更大的父代）的路由值：

```json
PUT my_index/_doc/3?routing=1&refresh   //1
{
  "text": "This is a vote",
  "my_join_field": {
    "name": "vote",
    "parent": "2"  //2
  }
}
```

> //1: 该子文档必须与其祖父母文档和父文档文档位于同一分片上
>
> //2: 该文档的父ID（必须指向答案文档）
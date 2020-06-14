# 布尔查询

​		与文档匹配的查询，这些文档与其他查询的布尔组合匹配。 布尔查询映射到Lucene BooleanQuery。 它是使用一个或多个布尔子句构建的，每个子句都具有类型的出现。 发生类型为：	

| 发生       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `must`     | 子句（查询）必须出现在匹配的文档中，并将有助于得分。         |
| `filter`   | 子句（查询）必须出现在匹配的文档中。 但是，与“必须”不同，查询的分数将被忽略。 过滤器子句在[过滤器上下文](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-filter-context.html)中执行，这意味着计分被忽略，并且考虑将子句用于缓存 。 |
| `should`   | 子句（查询）应出现在匹配的文档中。                           |
| `must_not` | 子句（查询）不得出现在匹配的文档中。 子句在[过滤器上下文](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-filter-context.html)中执行，这意味着计分被忽略，并且考虑将子句用于缓存。 由于计分被忽略，因此所有文档的分数都为“ 0”。 |



bool查询采用的是“更匹配是更好”的方法，因此每个匹配的must或should子句的得分将加在一起，以提供每个文档的最终_score。

```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```



## 1.使用minimum_should_match

​	您可以使用minimum_should_match参数指定返回的文档必须匹配的应当子句的数量或百分比。

如果布尔查询包含至少一个should子句，并且没有must或filter子句，则默认值为1。否则，默认值为0。

有关其他有效值，请参见[`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-minimum-should-match.html).



## 2.用bool.filter评分

​	在filter元素下指定的查询对得分没有影响-得分将返回0。得分仅受指定查询的影响。 例如，以下所有三个查询返回所有文档，其中状态字段包含术语“active”。

由于未指定评分查询，因此第一个查询为所有文档分配的分数为0：

```json
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

该布尔查询具有match_all查询，该查询为所有文档分配1.0的分数。

```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```



该constant_score查询的行为与上述第二个示例完全相同。 constant_score查询为该过滤器匹配的所有文档分配1.0的分数。

```json
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

## 3.使用命名查询查看匹配的子句
如果您需要知道布尔查询中的哪个子句与从查询返回的文档匹配，则可以使用 [named queries](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-request-body.html#request-body-search-queries-and-filters)为每个子句分配名称。












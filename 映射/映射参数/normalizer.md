# normalizer

关键字字段的规范化器属性与分析器类似，不同之处在于它保证分析链产生单个令牌。

在对keyword建立索引之前以及在通过诸如match查询之类的查询解析器或诸如术语查询之类的词级查询来搜索关键字字段时的搜索时，应用normalizer。



```json
PUT index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "char_filter": [],
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}

PUT index/_doc/1
{
  "foo": "BÀR"
}

PUT index/_doc/2
{
  "foo": "bar"
}

PUT index/_doc/3
{
  "foo": "baz"
}

POST index/_refresh

GET index/_search
{
  "query": {
    "term": {
      "foo": "BAR"
    }
  }
}

GET index/_search
{
  "query": {
    "match": {
      "foo": "BAR"
    }
  }
}
```

上面的查询匹配文档1和2，因为BÀR在索引和查询时间都转换为bar

```json
{
  "took": $body.took,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 2,
        "relation": "eq"
    },
    "max_score": 0.4700036,
    "hits": [
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.4700036,
        "_source": {
          "foo": "BÀR"
        }
      },
      {
        "_index": "index",
        "_type": "_doc",
        "_id": "2",
        "_score": 0.4700036,
        "_source": {
          "foo": "bar"
        }
      }
    ]
  }
}
```

同样，关键字在索引之前进行转换的事实也意味着聚合返回归一化值：

```json
GET index/_search
{
  "size": 0,
  "aggs": {
    "foo_terms": {
      "terms": {
        "field": "foo"
      }
    }
  }
}
```

返回：

```json
{
  "took": 43,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 3,
        "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "foo_terms": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "bar",
          "doc_count": 2
        },
        {
          "key": "baz",
          "doc_count": 1
        }
      ]
    }
  }
}
```










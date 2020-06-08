# _ignored字段

> 在6.4.0中添加。

_ignored字段建立索引并将每个字段的名称存储在文档中，因为该字段格式不正确且ignore_malformed已打开，因此已将其忽略。

该字段可与 [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-term-query.html), [`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-terms-query.html) and [`exists`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-exists-query.html)查询一起搜索，并作为搜索命中的一部分返回。

例如，以下查询匹配具有一个或多个字段被忽略的所有文档：

```json
GET _search
{
  "query": {
    "exists": {
      "field": "_ignored"
    }
  }
}
```

同样，以下查询查找在索引时间忽略@timestamp字段的所有文档：

```json
GET _search
{
  "query": {
    "term": {
      "_ignored": "@timestamp"
    }
  }
}
```


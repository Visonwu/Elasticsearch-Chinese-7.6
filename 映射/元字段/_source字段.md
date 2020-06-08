# _source 字段

_source字段包含在索引时间传递的原始JSON文档主体。 _source字段本身未编制索引（因此不可搜索），但已存储该字段，以便在执行获取请求（如get或search）时可以将其返回。

## 1.禁用_source字段
尽管非常方便，但是source字段确实会导致索引内的存储开销。 因此，可以如下禁用它：

```json
PUT tweets
{
  "mappings": {
    "_source": {
      "enabled": false
    }
  }
}
```

> 注：
>
> 禁用_source字段之前请三思
> 用户通常不考虑后果就禁用_source字段，然后后悔。 如果_source字段不可用，则不支持许多功能：
>
> - [`update`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-update.html), [`update_by_query`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-update-by-query.html), and [`reindex`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-reindex.html) API。
> - 实时突出显示。
> - 从一个Elasticsearch索引重新索引到另一索引的能力，以更改映射或分析，或将索引升级到新的主要版本。
> - 通过查看索引时使用的原始文档来调试查询或聚合的能力。
> - 将来有可能自动修复索引损坏。



>  提示：如果需要磁盘空间，则应提高压缩级别，而不要禁用_source。



> 指标用例
>
> 指标用例与其他基于时间或日志的用例不同，因为有许多小文档仅由数字，日期或关键字组成。 没有更新，没有突出显示请求，并且数据快速老化，因此无需重新编制索引。 搜索请求通常使用简单的查询按日期或标签过滤数据集，并且结果以聚合形式返回。
>
> 在这种情况下，禁用_source字段将节省空间并减少I / O。



## 2.包含/排除_source中的字段
仅限专家的功能是能够在对文档建立索引之后但在存储_source字段之前修剪_source字段的内容。

> 警示：从_source删除字段与禁用_source具有类似的缺点，尤其是您无法将文档从一个Elasticsearch索引重新索引到另一个的事实。 考虑改用源过滤。



包含/排除参数（也接受通配符）可以按如下方式使用：

```json
PUT logs
{
  "mappings": {
    "_source": {
      "includes": [
        "*.count",
        "meta.*"
      ],
      "excludes": [
        "meta.description",
        "meta.other.*"
      ]
    }
  }
}

PUT logs/_doc/1
{
  "requests": {
    "count": 10,
    "foo": "bar" 		//1
  },
  "meta": {
    "name": "Some metric",
    "description": "Some metric description", 	//1
    "other": {
      "foo": "one", 	//1
      "baz": "two" 		//1
    }
  }
}

GET logs/_search
{
  "query": {
    "match": {
      "meta.other.foo": "one" 	//2
    }
  }
}
```

> //1:这些字段将从存储的_source字段中删除。
>
>
> //2:即使它不在存储的_source中，我们仍然可以搜索该字段。
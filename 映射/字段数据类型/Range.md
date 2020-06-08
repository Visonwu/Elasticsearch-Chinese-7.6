# Range数据类型

支持以下范围类型：

| `integer_range` | 一系列为带符号的32位整数，最小值为-2的31幂，最大值为2的 31幂减1。 |
| --------------- | ------------------------------------------------------------ |
| `float_range`   | 一系列单精度32位IEEE 754浮点值。                             |
| `long_range`    | 一系列带符号的64位整数，最小值为-2的63幂，最大值为2的63幂减1。 |
| `double_range`  | 一系列双精度64位IEEE 754浮点值。                             |
| `date_range`    | 自系统时代以来经过的一系列日期值，表示为无符号的64位整数毫秒。 |
| `ip_range`      | 支持[IPv4](https://en.wikipedia.org/wiki/IPv4)或[IPv6](https://en.wikipedia.org/wiki/IPv6)（或混合）地址的一系列ip值。 |

下面是使用各种范围字段配置映射的示例，然后是为几种范围类型建立索引的示例。

```json
PUT range_index
{
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "expected_attendees": {
        "type": "integer_range"
      },
      "time_frame": {
        "type": "date_range", 		//1:
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}

PUT range_index/_doc/1?refresh
{
  "expected_attendees" : { 	//2:
    "gte" : 10,
    "lte" : 20
  },
  "time_frame" : { 			//3:
    "gte" : "2015-10-31 12:00:00",		//4: 
    "lte" : "2015-11-01"
  }
}
```

> //1: date_range类型接受由日期类型定义的相同字段参数。
>
>
> //2: 索引10至20位与会者的会议的示例。
>
>
> //3: 日期范围接受与日期范围查询中所述相同的格式。
>
>
> //4: 使用日期时间戳的日期范围示例。 这也接受日期数学格式。 请注意，在索引编制时不能使用“现在”。



以下是在名为“ expected_attendees”的integer_range字段上进行词条查询的示例。

```json
GET range_index/_search
{
  "query" : {
    "term" : {
      "expected_attendees" : {
        "value": 12
      }
    }
  }
}
```

以上查询产生的结果:

```json
{
  "took": 13,
  "timed_out": false,
  "_shards" : {
    "total": 2,
    "successful": 2,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "range_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "expected_attendees" : {
            "gte" : 10, "lte" : 20
          },
          "time_frame" : {
            "gte" : "2015-10-31 12:00:00", "lte" : "2015-11-01"
          }
        }
      }
    ]
  }
}
```

以下是在名为“ time_frame”的date_range字段上进行date_range查询的示例。

```json
GET range_index/_search
{
  "query" : {
    "range" : {
      "time_frame" : { 		//1:
        "gte" : "2015-10-31",
        "lte" : "2015-11-01",
        "relation" : "within" 
      }
    }
  }
}
```

> //1: 范围查询的作用与[range query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-range-query.html).中描述的相同。
>
>
> //2: 范围字段上的范围查询支持一个关系参数，该参数可以是WITHIN，CONTAINS，INTERSECTS（默认值）之一。

该查询产生类似的结果：

```json
{
  "took": 13,
  "timed_out": false,
  "_shards" : {
    "total": 2,
    "successful": 2,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "range_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "expected_attendees" : {
            "gte" : 10, "lte" : 20
          },
          "time_frame" : {
            "gte" : "2015-10-31 12:00:00", "lte" : "2015-11-01"
          }
        }
      }
    ]
  }
}
```



## 1.IP Range

除上述范围格式外，IP范围还可以CIDR表示法提供：

```json
PUT range_index/_mapping
{
  "properties": {
    "ip_whitelist": {
      "type": "ip_range"
    }
  }
}

PUT range_index/_doc/2
{
  "ip_whitelist" : "192.168.0.0/16"
}
```



## 2. Range字段的参数
range 类型接受以下参数：

[`coerce`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/coerce.html): 	尝试将字符串转换为数字，并截断整数部分。 接受true（默认）和false。

[`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html): 	 映射字段级查询时间的增加。 接受浮点数，默认为1.0。

[`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html): 	该字段是否可以搜索？ 接受true（默认）和false。

[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html):	 是否应与_source字段分开存储和检索字段值。 接受true或false（默认）。


# IP数据类型

ip字段可以索引/存储IPv4或IPv6地址。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "ip_addr": {
        "type": "ip"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "ip_addr": "192.168.1.1"
}

GET my_index/_search
{
  "query": {
    "term": {
      "ip_addr": "192.168.0.0/16"
    }
  }
}
```

注：您还可以使用ip_range数据类型将ip范围存储在单个字段中。



## 1.ip字段的参数
ip字段接受以下参数：

| [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-boost.html) | 映射字段级查询时间的增加。 接受浮点数，默认为1.0。           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/doc-values.html) | 该字段是否应该以column-stride的方式存储在磁盘上，以便以后可以用于排序，聚合或编写脚本？ 接受true（默认）或false。 |
| [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-index.html) | 该字段是否可以搜索？ 接受“ true”（默认）和“ false”.          |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/null-value.html) | 接受一个IPv4值，该值将替换任何显式的“ null”值。 默认为“ null”，这意味着该字段被视为丢失。 |
| [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-store.html) | 字段值是否应与[`_source`](https_www.elastic.co/guide/en/elasticsearch/reference/7.6/mapping-source-field.html)字段分开存储和检索。 接受“ true”或“ false”（默认）。 |

## 2. 查询ip字段
查询IP地址最常用的方法是使用CIDR表示法：[ip_address] / [prefix_length]。 例如：

```json
GET my_index/_search
{
  "query": {
    "term": {
      "ip_addr": "192.168.0.0/16"
    }
  }
}
```

或者

```json
GET my_index/_search
{
  "query": {
    "term": {
      "ip_addr": "2001:db8::/48"
    }
  }
}
```

还要注意，冒号是query_string查询的特殊字符，因此将需要转义ipv6地址。 最简单的方法是将引号放在搜索值的周围：

```json
GET my_index/_search
{
  "query": {
    "query_string" : {
      "query": "ip_addr:\"2001:db8::/48\""
    }
  }
}
```
















**Dense vector数据类型**



`density_vector`字段存储浮点值的密集向量。 向量中可以包含的最大维数不应超过2048。density_vector字段是单值字段。

这些向量可用于文档评分。 例如，[文档分数](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-script-score-query.html#vector-functions)可以表示给定查询向量和索引文档向量之间的距离。

您将密集向量索引为浮点数数组。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "dense_vector",
        "dims": 3    	//1
      },
      "my_text" : {
        "type" : "keyword"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "my_text" : "text1",
  "my_vector" : [0.5, 10, 6]
}

PUT my_index/_doc/2
{
  "my_text" : "text2",
  "my_vector" : [-0.5, 10, 10]
}
```

> //1: dims-矢量中的维数，必需的参数。

​	在内部，每个文档的密集向量都被编码为二进制doc值。 它的大小（以字节为单位）等于4 * dims + 4，其中dims是矢量尺寸的数量。
# [Sparse vector](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/sparse-vector.html) 数据类型

> 注：在7.6中弃用。
> sparse_vector类型已弃用，并将在8.0中删除。



sparse_vector字段存储浮点值的稀疏向量。 向量中的最大维数不能超过1024。文档之间的维数可以不同。 sparse_vector字段是单值字段。

这些向量可用于文档评分。 例如，文档分数可以表示给定查询向量和索引文档向量之间的距离。

您将一个稀疏向量表示为一个对象，其中对象字段是维，而字段值是这些维的值。 维度是从0到65535的整数值，被编码为字符串。 尺寸不需要按顺序排列。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_vector": {
        "type": "sparse_vector"
      },
      "my_text" : {
        "type" : "keyword"
      }
    }
  }
}
```

```json
PUT my_index/_doc/1
{
  "my_text" : "text1",
  "my_vector" : {"1": 0.5, "5": -0.5,  "100": 1}
}

PUT my_index/_doc/2
{
  "my_text" : "text2",
  "my_vector" : {"103": 0.5, "4": -0.5,  "5": 1, "11" : 1.2}
}
```

在内部，每个文档的稀疏矢量都被编码为二进制doc值。 其字节大小等于6 * NUMBER_OF_DIMENSIONS + 4，其中NUMBER_OF_DIMENSIONS-向量的维数。




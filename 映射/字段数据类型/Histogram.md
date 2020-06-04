# Histogram 数据类型

用于存储表示直方图的预聚合数值数据的字段。 该数据是使用两个成对的数组定义的：

> - 一个双精度值的值数组，表示直方图的存储桶。 这些值必须按升序提供。
> - 对应的整数计数数组代表每个桶中有多少个值。 这些数字必须为正或零。



因为values数组中的元素与count数组相同位置的元素相对应，所以这两个数组必须具有相同的长度。

> 注意：
>
> - 直方图字段只能存储一对值，并且每个文档都可以计数数组。 不支持嵌套数组。
> - 直方图字段不支持排序。



## 1.用途

`histogram`字段主要用于聚合。 为了使汇总更易于访问，直方图字段数据存储为二进制doc值，但未编制索引。 它的字节大小最多为13 * numValues，其中numValues是提供的数组的长度。

由于未对数据建立索引，因此只能将直方图字段用于以下聚合和查询：

> - [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-metrics-percentile-aggregation.html) aggregation
> - [percentile ranks](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-metrics-percentile-rank-aggregation.html) aggregation
> - [exists](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-exists-query.html) query



## 2.建立直方图

​		当使用直方图作为聚合的一部分时，结果的准确性将取决于直方图的构造方式。 重要的是要考虑将用于构建百分比聚合模式。 一些可能性包括：



> - 对于T-摘要模式，values数组代表平均质心位置，而counts数组代表归属于每个质心的值的数量。 如果算法已经开始近似百分位数，则此误差会在直方图中继续。
> - 对于高动态范围（HDR）直方图模式，values数组表示每个存储段间隔的固定上限，而counts数组表示归因于每个间隔的值的数量。 此实现保持固定的最坏情况百分比误差（指定为有效数字的数量），因此，生成直方图时使用的值将是您在聚合时可以达到的最大精度。



直方图字段是“算法不可知”的字段，不存储特定于T-Digest或HDRHistogram的数据。 虽然这意味着该字段可以从技术上与这两种算法进行汇总，但实际上用户应选择一种算法并以这种方式索引数据（例如T-Digest的质心或HDRHistogram的间隔）以确保最佳准确性。



## 3. 例子

以下创建索引API请求创建具有两个字段映射的新索引：

> - my_histogram，用于存储百分位数据的直方图字段
> - my_text，用于存储直方图标题的关键字字段

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "my_histogram": {
        "type" : "histogram"
      },
      "my_text" : {
        "type" : "keyword"
      }
    }
  }
}
```

以下索引API请求存储为两个直方图预先聚合的：histogram_1和histogram_2。

```json
PUT my_index/_doc/1
{
  "my_text" : "histogram_1",
  "my_histogram" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5],   //1
      "counts" : [3, 7, 23, 12, 6] 				//2
   }
}

PUT my_index/_doc/2
{
  "my_text" : "histogram_2",
  "my_histogram" : {	
      "values" : [0.1, 0.25, 0.35, 0.4, 0.45, 0.5], 	//1
      "counts" : [8, 17, 8, 7, 6, 2] 					//2
   }
}
```



> 每个存储桶的值。 数组中的值被视为双精度值，并且必须以递增顺序给出。 对于[T型摘要](<https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-approximation>)直方图，此值表示平均值。 在HDR直方图的情况下，这表示迭代的值。

>  数每个桶。 数组中的值被视为整数，并且必须为正数或零。 负值将被拒绝。 存储桶与计数之间的关系由数组中的位置给出。


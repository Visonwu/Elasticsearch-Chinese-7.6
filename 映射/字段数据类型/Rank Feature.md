# Rank Feature 字段类型

`rank_feature` 可以为数字编制索引，以便以后可以在使用`rank_feature` 查询的查询中增强数字。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "pagerank": {
        "type": "rank_feature" 		//1
      },
      "url_length": {
        "type": "rank_feature",
        "positive_score_impact": false 	//2
      }
    }
  }
}

PUT my_index/_doc/1
{
  "pagerank": 8,
  "url_length": 22
}

GET my_index/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank"
    }
  }
}
```

> //1: 等级要素字段必须使用rank_feature字段类型
>
>
> //2: 与得分负相关的排名特征需要声明



> 注：
>
> rank_feature字段仅支持单值字段和严格为正的值。 多值字段和负值将被拒绝。
>
> rank_feature字段不支持查询，排序或聚合。 它们只能在rank_feature查询中使用。
>
> rank_feature字段仅保留9个有效位用于精度，这意味着约0.4％的相对误差。



与得分负相关的排名特征应将positive_score_impact设置为false（默认为true）。 rank_feature查询将使用它来修改评分公式，以使分数随着要素的值而减少而不是增加。 例如，在Web搜索中，URL长度是常用的功能，它与分数负相关。
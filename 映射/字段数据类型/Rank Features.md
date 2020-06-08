# Rank Features 字段类型

rank_features字段可以为数字特征向量建立索引，以便以后可以将它们用于在具有rank_feature查询的查询中增强文档。

它类似于rank_feature数据类型，但更适合于特征列表稀疏的情况，因此为每个特征映射添加一个字段都不合理。

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "topics": {
        "type": "rank_features" 		//1 
      }
    }
  }
}

PUT my_index/_doc/1
{
  "topics": { 				//2
    "politics": 20,
    "economics": 50.8
  }
}

PUT my_index/_doc/2
{
  "topics": {
    "politics": 5.2,
    "sports": 80.1
  }
}

GET my_index/_search
{
  "query": {
    "rank_feature": {
      "field": "topics.politics"
    }
  }
}
```

> Rank features 字段必须使用rank_features字段类型
>
>
> Rank features 字段必须是带有字符串键和严格正数的哈希值



> 注：
>
> rank_features字段仅支持单值特征和严格为正的值。 多值字段以及零或负值将被拒绝。
>
> rank_features字段不支持排序或汇总，只能使用rank_feature查询来查询。
>
> rank_features字段仅保留9个有效位用于精度，这意味着约0.4％的相对误差。
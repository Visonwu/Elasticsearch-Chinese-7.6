# Function score query
function_score允许您修改查询检索的文档分数。 例如，如果分数函数在计算上很昂贵，并且足以在过滤后的文档集上计算分数，则此功能很有用。

要使用function_score，用户必须定义一个查询和一个或多个函数，这些函数为查询返回的每个文档计算一个新分数。

function_score与以下一个函数一起使用：

```json
GET /_search
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 			//1
            "boost_mode":"multiply"
        }
    }
}
```

> //1: 有关支持的功能列表，请参见[Function score](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-function-score-query.html#score-functions) 。



此外，可以组合几个功能。 在这种情况下，可以选择仅在文档与给定的过滤查询匹配时才应用功能

```json
GET /_search
{
    "query": {
        "function_score": {
          "query": { "match_all": {} },
          "boost": "5", 		//1
          "functions": [
              {
                  "filter": { "match": { "test": "bar" } },
                  "random_score": {}, 	//2
                  "weight": 23
              },
              {
                  "filter": { "match": { "test": "cat" } },
                  "weight": 42
              }
          ],
          "max_boost": 42,
          "score_mode": "max",
          "boost_mode": "multiply",
          "min_score" : 42
        }
    }
}
```

> //1: 提高整个查询。
>
>
> //2: 有关支持的功能列表，请参见功能得分。



> 注：
>
> 每个函数的过滤查询所产生的分数并不重要。



如果没有给函数提供过滤器，则等效于指定“ match_all”：{}

首先，通过定义的功能对每个文档评分。 参数score_mode指定如何组合计算的分数：










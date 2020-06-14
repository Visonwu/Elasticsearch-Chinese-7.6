# Boost query
返回匹配肯定查询的文档，同时降低也匹配否定查询的文档的相关性得分。

您可以使用提升查询来降级某些文档，而不必将它们从搜索结果中排除。

 

例子：

```json
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```



用于提升的顶级参数
**positive**: （必需的查询对象）要运行的查询。 返回的所有文档都必须与此查询匹配。
**negative**: （必需的查询对象）查询用于降低匹配文档的相关性得分。



如果返回的文档与positive查询和该查询匹配，则增强查询将计算该文档的最终相关性得分，如下所示：

- 从肯定查询中获取原始的相关性分数。
- 将分数乘以negative_boost值。
   

**`negative_boost`**: （必需，浮点数）0到1.0之间的浮点数，用于降低与否定查询匹配的文档的相关性得分。
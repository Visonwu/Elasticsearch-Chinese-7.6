##  Constant score query

​		包装过滤查询，并返回相关分数等于boost参数值的每个匹配文档。

```json
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}

```

## constant_score的顶级参数
**filter**
（必需的查询对象）过滤要运行的查询。 返回的所有文档都必须与此查询匹配。

过滤查询不计算相关性分数。 为了提高性能，Elasticsearch自动缓存经常使用的过滤器查询。



**boost:**（可选，float）浮点数，用作与过滤器查询匹配的每个文档的恒定相关性得分。 默认为1.0。
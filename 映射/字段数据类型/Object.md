# Object 数据类型

JSON文档本质上是分层的：文档可能包含内部对象，而内部对象又可能包含内部对象本身：

```json
PUT my_index/_doc/1
{ 							//1
  "region": "US",
  "manager": { 				//2
    "age":     30,
    "name": { 				//3
      "first": "John",
      "last":  "Smith"
    }
  }
}
```

> //1: 外部文档也是JSON对象。
>
>
> //2: 它包含一个称为管理器的内部对象。
>
>
> //3: 依次包含一个名为name的内部对象。

在内部，此文档被索引为键-值对的简单平面列表，如下所示：

```json
{
  "region":             "US",
  "manager.age":        30,
  "manager.name.first": "John",
  "manager.name.last":  "Smith"
}
```

上面文档的显式映射如下所示：

```json
PUT my_index
{
  "mappings": {
    "properties": {  			 //1
      "region": {
        "type": "keyword"
      },
      "manager": { 				//2
        "properties": {
          "age":  { "type": "integer" },
          "name": { 			//3
            "properties": {
              "first": { "type": "text" },
              "last":  { "type": "text" }
            }
          }
        }
      }
    }
  }
}
```

> //1: 顶级映射定义中的属性。
>
>
> //2: 管理者字段是内部对象字段。
>
>
> //3: manager.name字段是manager字段内的内部对象字段。
>

您不需要将字段类型显式设置为对象，因为这是默认值。



## 1.对象字段的参数
对象字段接受以下参数：

| [`dynamic`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/dynamic.html) | 是否应将新的“属性”动态添加到现有对象。 接受“ true”（默认），“ false”和“ strict”。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`enabled`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/enabled.html) | 为对象字段指定的JSON值是应该解析和索引（默认为true）还是完全忽略（false）。 |
| [`properties`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/properties.html) | 对象中的字段可以是任何[数据类型](https://www.elastic.co/guide/zh-cn/elasticsearch/reference/7.6/mapping-types.html)，包括“对象”。 可以将新属性添加到现有对象。 |

> 注：如果您需要索引对象数组而不是单个对象的索引，请先阅读嵌套nested 数据类型。
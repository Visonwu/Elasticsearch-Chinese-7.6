# coerce

数据并不总是干净的。 根据产生方式的不同，数字可能会在JSON主体中呈现为真实的JSON数字，例如 5，但也可能呈现为字符串，例如 “ 5”。 或者，应将应为整数的数字呈现为浮点数，例如 5.0，甚至是“ 5.0”。



coerce强制尝试清除脏值以适合字段的数据类型。 例如：

- 字符串将被强制转换为数字。
- 浮点将被截断为整数值。



举例：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "number_one": {
        "type": "integer"
      },
      "number_two": {
        "type": "integer",
        "coerce": false
      }
    }
  }
}

PUT my_index/_doc/1
{
  "number_one": "10" 		//1
}

PUT my_index/_doc/2
{
  "number_two": "10" 		//2
}
```

>//1: number_one字段将包含整数10。
>
>//2: 由于禁用了强制，因此该文档将被拒绝。



> 注：可以使用PUT映射API在现有字段上更新强制设置值。



## 1.索引级默认
可以在索引级别上设置index.mapping.coerce设置，以在所有映射类型中全局禁用强制：

```json
PUT my_index
{
  "settings": {
    "index.mapping.coerce": false
  },
  "mappings": {
    "properties": {
      "number_one": {
        "type": "integer",
        "coerce": true
      },
      "number_two": {
        "type": "integer"
      }
    }
  }
}

PUT my_index/_doc/1
{ "number_one": "10" } 		//1

PUT my_index/_doc/2
{ "number_two": "10" } 		//2
```

> //1: number_one字段将覆盖索引级别设置以启用强制。
>
>
> //2: 该文档将被拒绝，因为number_two字段继承了索引级强制设置。
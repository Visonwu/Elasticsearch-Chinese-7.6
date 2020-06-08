# dynamic

默认情况下，仅通过索引包含新字段的文档，即可将字段动态添加到文档或文档中的内部对象。 例如：

```json
PUT my_index/_doc/1 			//1
{	
  "username": "johnsmith",
  "name": {
    "first": "John",
    "last": "Smith"
  }
}

GET my_index/_mapping 		//2

PUT my_index/_doc/2 		//3
{
  "username": "marywhite",
  "email": "mary@white.com",
  "name": {
    "first": "Mary",
    "middle": "Alice",
    "last": "White"
  }
}

GET my_index/_mapping 		//4
```

> //1: 本文档介绍了字符串字段用户名，对象字段名称以及名称对象下的两个字符串字段，可以将其称为name.first和name.last。
>
>
> //2: 检查映射以验证以上内容。
>
>
> //3: 本文档添加了两个字符串字段：email和name.middle。
>
>
> /4: 检查映射以验证更改。



 [*Dynamic Mapping*](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/dynamic-mapping.html)中介绍了如何检测新字段并将其添加到映射的详细信息。

动态设置控制是否可以动态添加新字段。 它接受三种设置：



| `true`   | 新检测到的字段将添加到映射中。 （默认）                      |
| -------- | ------------------------------------------------------------ |
| `false`  | 新检测到的字段将被忽略。 这些字段将不会被索引，因此将无法搜索，但仍会出现在返回匹配的`_source'字段中。 这些字段不会添加到映射中，必须显式添加新字段。 |
| `strict` | 如果检测到新字段，则会引发异常并拒绝文档。 必须将新字段显式添加到映射中。 |



动态设置可以在映射类型级别和每个内部对象上设置。 内部对象从其父对象或映射类型继承设置。 例如：

```json
PUT my_index
{
  "mappings": {
    "dynamic": false, 		//1
    "properties": {
      "user": { 			//2
        "properties": {
          "name": {
            "type": "text"
          },
          "social_networks": { //3
            "dynamic": true,
            "properties": {}
          }
        }
      }
    }
  }
}
```

> //1: 在类型级别禁用动态映射，因此不会动态添加新的顶级字段。
>
>
> //2: 用户对象继承类型级别设置。
>
>
> //3: user.social_networks对象启用动态映射，因此可以将新字段添加到此内部对象。



> 注：
>
> 可以使用PUT映射API在现有字段上更新动态设置。








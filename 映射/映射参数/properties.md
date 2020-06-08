# properties

类型映射， [`object` fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/object.html)字段和 [`nested` fields](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/nested.html)字段包含称为属性的子字段。 这些属性可以是任何数据类型，包括object和nested。 可以添加属性：

- 通过在创建索引时定义它们来明确地定义它们。
- 通过在使用PUT映射API添加或更新映射类型时定义它们来明确定义它们。
- 仅通过索引包含新字段的文档即可动态地实现。



下面是将属性添加到映射类型，对象字段和嵌套字段的示例：

```json
PUT my_index
{
  "mappings": {
    "properties": { 			//1
      "manager": {
        "properties": { 		//2
          "age":  { "type": "integer" },
          "name": { "type": "text"  }
        }
      },
      "employees": {
        "type": "nested",
        "properties": { 		//3
          "age":  { "type": "integer" },
          "name": { "type": "text"  }
        }
      }
    }
  }
}

PUT my_index/_doc/1 			//4
{
  "region": "US",
  "manager": {
    "name": "Alice White",
    "age": 30
  },
  "employees": [
    {
      "name": "John Smith",
      "age": 34
    },
    {
      "name": "Peter Brown",
      "age": 26
    }
  ]
}
```

> //1: 顶级映射定义中的属性。
>
>
> //2: 管理器对象字段下的属性。
>
>
> //3: 员工嵌套字段下的属性。
>
>
> //4: 与上述映射相对应的示例文档。



> 对于同一索引中相同名称的字段，属性设置可以具有不同的设置。 可以使用PUT映射API将新属性添加到现有字段。



## 点表示法
内部字段可以在查询，聚合等中使用点表示法进行引用：

```json
GET my_index/_search
{
  "query": {
    "match": {
      "manager.name": "Alice White"
    }
  },
  "aggs": {
    "Employees": {
      "nested": {
        "path": "employees"
      },
      "aggs": {
        "Employee Ages": {
          "histogram": {
            "field": "employees.age",
            "interval": 5
          }
        }
      }
    }
  }
}
```

> 必须指定内部字段的完整路径。








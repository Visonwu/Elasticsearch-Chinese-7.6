Elasticsearch提供了基于JSON的完整查询DSL（特定于域的语言）来定义查询。 将查询DSL视为查询的AST（抽象语法树），它由两种子句组成：

- **叶子查询子句**
  叶子查询子句在特定字段中查找特定值，例如[`match`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-match-query.html), [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-term-query.html)或[`range`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-range-query.html)查询。 这些查询可以自己使用。
- **复合查询子句**
  复合查询子句包装其他叶查询或复合查询，并用于以逻辑方式组合多个查询（例如 [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-bool-query.html)或 [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-dis-max-query.html)查询），或更改其行为（例如 [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-constant-score-query.html)查询）。



查询子句的行为会有所不同，具体取决于它们是在查询上下文中还是在过滤器上下文中使用。






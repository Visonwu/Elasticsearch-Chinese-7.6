# norms

规范存储各种归一化因子，这些归一化因子随后在查询时使用，以便计算相对于查询的文档分数。

尽管评分非常有用，但规范也需要大量磁盘（通常对于索引中每个字段，每个文档一个字节的顺序，即使对于没有此特定字段的文档也是如此）。 因此，如果您不需要在特定字段上评分，则应禁用该字段上的规范。 特别是对于仅用于过滤或聚合的字段就是这种情况。



> 可以使用PUT映射API在现有字段上禁用规范。

可以使用PUT映射API禁用规范（但事后不能重新启用），如下所示：

```
PUT my_index/_mapping
{
  "properties": {
    "title": {
      "type": "text",
      "norms": false
    }
  }
}
```



> Norms不会立即删除，但是随着您继续为新文档建立索引，旧段将合并到新段中，因此将删除这些Norms。 由于某些文档将不再具有规范，而其他文档可能仍具有规范，因此在删除了规范的字段上进行的任何分数计算都可能返回不一致的结果。








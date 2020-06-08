# _field_names字段

_field_names字段用于索引文档中包含非null值的每个字段的名称。 存在查询使用此字段查找特定字段具有或不具有任何非空值的文档。

现在，_field_names字段仅索引禁用了doc_values和norm的字段的名称。 对于启用了doc_values或norm的字段，存在查询将仍然可用，但不会使用_field_names字段。



## 1.禁用_field_names
>  禁用_field_names已被弃用，并将在以后的主要版本中删除。



通常不必禁用\_field_names，因为它不再像以前那样承载索引开销。 如果您有很多禁用了doc_values和norm的字段，并且不需要使用这些字段执行exists查询，则可能需要通过将以下内容添加到映射中来禁用_field_names：

```json
PUT tweets
{
  "mappings": {
    "_field_names": {
      "enabled": false
    }
  }
}
```


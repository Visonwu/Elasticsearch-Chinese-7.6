# analyzer

> 注：仅文本字段支持分析器映射参数。



Analyzer参数指定索引或搜索文本字段时用于 [text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis.html) 的分析器。

除非使用 [`search_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-analyzer.html)映射参数覆盖，否则此分析器将用于[index and search analysis](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis-index-search-time.html).。 请参阅[Specify an analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/specify-analyzer.html)。

> 我们建议在生产中使用分析仪之前对其进行测试。 请参阅测试分析仪。



## 1.search_quote_analyzer
search_quote_analyzer设置允许您为短语指定分析器，这在处理禁用短语查询的停用词时特别有用。

要禁用短语的停用词，需要使用三个分析器设置的字段：

- 1.分析器设置，用于索引包括停用词在内的所有术语

- 2.非短语查询的search_analyzer设置，该设置将删除停用词
- 3.短语查询的search_quote_analyzer设置不会删除停用词

```json
PUT my_index
{
   "settings":{
      "analysis":{
         "analyzer":{
            "my_analyzer":{ 					//1
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase"
               ]
            },
            "my_stop_analyzer":{ 				//2
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "english_stop"
               ]
            }
         },
         "filter":{
            "english_stop":{
               "type":"stop",
               "stopwords":"_english_"
            }
         }
      }
   },
   "mappings":{
       "properties":{
          "title": {
             "type":"text",
             "analyzer":"my_analyzer", 				//3
             "search_analyzer":"my_stop_analyzer", 	//4
             "search_quote_analyzer":"my_analyzer" 	//5
         }
      }
   }
}

PUT my_index/_doc/1
{
   "title":"The Quick Brown Fox"
}

PUT my_index/_doc/2
{
   "title":"A Quick Brown Fox"
}

GET my_index/_search
{
   "query":{
      "query_string":{
         "query":"\"the quick brown fox\"" 		//6
      }
   }
}
```

> //1: my_analyzer分析器，可标记包括停用词在内的所有术语
>
>
> //2: my_stop_analyzer分析器，可删除停用词
>
>
> //3: 指向my_analyzer分析器的分析器设置，将在索引时间使用
>
>
> //4 : 指向my_stop_analyzer的search_analyzer设置，并删除非短语查询的停用词
>
>
> //5: search_quote_analyzer设置指向my_analyzer分析器，并确保不从短语查询中删除停用词
>
>
> //6: 由于查询是用引号引起来的，因此它被检测为短语查询，因此search_quote_analyzer会插入并确保不将停用词从查询中删除。然后，my_analyzer分析器将返回与这些文档之一匹配的以下令牌[the，quick，brown，fox]。同时，将使用my_stop_analyzer分析器分析术语查询，该分析器会过滤掉停用词。因此，搜索The quick brown fox或A quick brown fox都会返回两个文档，因为两个文档都包含以下标记[quick，brown，fox]。没有search_quote_analyzer，就不可能对短语查询进行精确匹配，因为短语查询中的停用词将被删除，导致两个文档都匹配。
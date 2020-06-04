# Geo-point 数据类型

geo_point类型的字段接受纬度-经度对，可以使用以下：

> - 在[边界框](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-geo-bounding-box-query.html)内，中心点一定[距离](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-geo-distance-query.html)内或[多边形](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-geo-polygon-query.html)内找到地理点。
> - 以[地理位置](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-aggregations-bucket-geohashgrid-aggregation.html)或距中心点的距离汇总文档。
> - 将距离整合到文档的[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-function-score-query.html)中。
> - 按距离对文档进行[排序](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-request-body.html#geo-sorting)。

可以通过五种方式指定地理位置，如下所示：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT my_index/_doc/1
{
  "text": "Geo-point as an object",
  "location": { 		//1
    "lat": 41.12,
    "lon": -71.34
  }
}

PUT my_index/_doc/2
{
  "text": "Geo-point as a string",
  "location": "41.12,-71.34" 	//2
}

PUT my_index/_doc/3
{
  "text": "Geo-point as a geohash",
  "location": "drm3btev3e86" 	//3
}

PUT my_index/_doc/4
{
  "text": "Geo-point as an array",
  "location": [ -71.34, 41.12 ] //4
}

PUT my_index/_doc/5
{
  "text": "Geo-point as a WKT POINT primitive",
  "location" : "POINT (-71.34 41.12)" //5
}

GET my_index/_search
{
  "query": {
    "geo_bounding_box": { 	//6
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -72
        },
        "bottom_right": {
          "lat": 40,
          "lon": -74
        }
      }
    }
  }
}
```

> //1: 表示为对象的地理点，具有lat和lon键。
>
>
> //2: 以字符串形式表示的地理点，格式为“ lat，lon”。
>
>
> //3: 表示为哈希的地理点。
>
>
> //4:用数组表示的地理点，其格式为：[lon，lat]
>
>
> //5:地理点，表示为格式为“ POINT（lon lat）”的知名文本POINT
>
>
> //6:地理边界框查询，查找位于该框内的所有地理点。



> 重要：表示为数组或字符串的地理位置
>
> - 请注意，字符串地理位置按lat，lon排序，而数组地理位置按相反的顺序：lon，lat。
>
> - 最初，lat，lon用于数组和字符串，但是数组格式很早就更改为与GeoJSON所使用的格式一致。



> 注意：
>
> 一个点可以表示为[geohash](http://en.wikipedia.org/wiki/Geohash)。 Geohash是交错的纬度和经度的位的base32编码的字符串。 geohash中的每个字符都会增加5位精度。 因此，哈希越长，它越精确。 为了进行索引，将目标哈希值转换为纬度-经度对。 在此过程中，仅使用前12个字符，因此在geohash中指定超过12个字符不会提高精度。 这12个字符提供60位，这应该将可能的错误减少到2cm以下。



## 1.Geo-point的参数

geo_point字段接受以下参数：

| [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/ignore-malformed.html) | 如果为true，则会忽略格式错误的地理位置。 如果为“ false”（默认），则格式错误的地理位置会引发异常并拒绝整个文档。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ignore_z_value`                                             | 如果为“ true”（默认），则将接受三个维度点（存储在源代码中），但仅对经度和纬度值建立索引； 第三维被忽略。 如果为false，则包含除纬度和经度（二维）值以外的值的地理点将引发异常并拒绝整个文档。 |
| [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/null-value.html) | 接受一个替换任何显式“ null”值的geopoint值。 默认为“ null”，这意味着该字段被视为丢失。 |



## 2. 在脚本中使用Geo-point

在脚本中访问地理位置的值时，该值作为GeoPoint对象返回，该对象允许分别访问.lat和.lon值：

```javascript
def geopoint = doc['location'].value;
def lat      = geopoint.lat;
def lon      = geopoint.lon;
```

出于性能原因，最好直接访问纬度/经度值：

```javascript
def lat      = doc['location'].lat;
def lon      = doc['location'].lon;
```


# Geo-Shape 数据类型 

geo_shape数据类型有助于索引和搜索任意地理形状，例如矩形和多边形。 当正在索引的数据或正在执行的查询包含除点以外的其他形状时，应使用它。

您可以使用 [geo_shape Query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-geo-shape-query.html)此类型查询文档。



## 1. 映射选项

​		geo_shape映射将geo_json几何对象映射到geo_shape类型。 要启用它，用户必须将字段显式映射到geo_shape类型。



| Option               | Description                                                  | Default     |
| -------------------- | ------------------------------------------------------------ | ----------- |
| `tree`               | [6.6]在6.6中弃用。 不再使用PrefixTrees要使用的PrefixTree实现的名称：对于GeohashPrefixTree是`geohash`，对于QuadPrefixTree是`quadtree`。 注意：此参数仅与term和递归策略有关。 | `quadtree`  |
| `precision`          | [6.6]在6.6中弃用。 不再使用PrefixTrees此参数可以代替`tree_levels`来为`tree_levels`参数设置适当的值。 该值指定所需的精度，Elasticsearch将计算最佳的tree_levels值以兑现该精度。 该值应为数字，后跟可选的距离单位。 有效距离单位包括：“英寸”，“英寸”，“ yd”，“码”，“ mi”，“英里”，“ km”，“公里”，“ m”，“米”，“ cm”，“ 厘米，毫米，毫米。 注意：此参数仅与term和递归策略有关。 | `50m`       |
| `tree_levels`        | [6.6]在6.6中弃用。 不再使用PrefixTrees PrefixTree将使用的最大层数。 这可用于控制形状表示的精度，从而控制索引多少项。 默认为所选PrefixTree实现的默认值。 由于此参数要求对底层实现有一定程度的了解，因此用户可以改用“ precision”参数。 但是，Elasticsearch仅在内部使用tree_levels参数，即使您使用precision参数，这也是通过映射API返回的结果。 注意：该参数仅与term和递归策略有关。 | various     |
| `strategy`           | [6.6]在6.6中弃用。 不再使用PrefixTrees策略参数定义了如何在索引和搜索时表示形状的方法。 它还会影响可用的功能，因此建议让Elasticsearch自动设置此参数。 有两种可用的策略：“递归”和“ term”。 递归和术语策略已弃用，在以后的版本中将被删除。 虽然它们仍然可用，但Term策略仅支持点类型（points_only参数将自动设置为true），而递归策略则支持所有形状类型。 （重要说明：有关这些策略的更多详细信息，请参见[前缀树](https://www.elastic.co/guide/zh/elasticsearch/reference/7.6/geo-shape.html#prefix-trees)) | `recursive` |
| `distance_error_pct` | [6.6]在6.6中弃用。 不再使用PrefixTrees用作PrefixTree关于其精度的提示。 默认值为0.025（2.5％），最大支持值为0.5。 性能注意：如果显式定义了“ precision”或“ tree_level”定义，则该值默认为0。 这样可以保证映射中定义的级别的空间精度。 这会导致具有低误差的高分辨率形状（例如，误差小于0.001的1m处的大形状）大量使用内存。 为了提高索引性能（以查询准确性为代价），请使用合理的“ distance_error_pct”显式定义“ tree_level”或“ precision”，并注意较大的形状将具有更大的误报率。 注意：该参数仅与term和递归策略有关。 | `0.025`     |
| `orientation`        | （可选）定义如何解释多边形/多多边形的顶点顺序。 此参数定义两个坐标系规则（右手或左手）之一，可以用三种不同的方式来指定每个规则。 1.右手规则：`right`，`ccw`，`逆时针`，2。左手规则：`left`，`cw`，`顺时针`。 默认方向（“逆时针方向”）符合OGC标准，该标准以逆时针顺序定义外环顶点，而内环顶点（孔）以顺时针顺序定义。 在geo_shape映射中设置此参数可显式设置geo_shape字段的坐标列表的顶点顺序，但可以在每个单独的GeoJSON或WKT文档中覆盖。 | `ccw`       |
| `points_only`        | [6.6]在6.6中弃用。 不再使用PrefixTrees将此选项设置为“ true”（默认为“ false”）仅为点形状配置“ geo_shape”字段类型（注意：尚不支持多点）。 当已知只有点将被索引时，这可以优化“ geohash”和“ quadtree”的索引和搜索性能。 目前，无法在“ geo_point”字段类型上执行geo_shape查询。 此选项通过改善geo_shape字段上的点性能来弥合差距，从而使geo_shape查询在仅单点字段上是最佳的。 | `false`     |
| `ignore_malformed`   | 如果为true，则会忽略格式错误的GeoJSON或WKT形状。 如果为假（默认），则格式错误的GeoJSON和WKT形状会引发异常并拒绝整个文档。 | `false`     |
| `ignore_z_value`     | 如果为“ true”（默认），则将接受三个维度点（存储在源代码中），但仅对经度和纬度值建立索引； 第三维被忽略。 如果为false，则包含除纬度和经度（二维）值以外的值的地理点将引发异常并拒绝整个文档。 | `true`      |
| `coerce`             | 如果为true，则多边形中未闭合的线性环将自动闭合。             | `false`     |

## 2. 索引方法

​	通过将形状分解为三角形网格并将每个三角形索引为BKD树中的7维点来索引GeoShape类型。 这提供了近乎完美的空间分辨率（低至1e-7十进制精度），因为所有空间关系都是使用原始形状的编码矢量表示代替前缀树索引方法所使用的栅格网格表示来计算的。 镶嵌器的性能主要取决于定义多边形/多面体的顶点数量。 尽管这是默认的索引技术，但仍可以通过根据适当的“映射选项”设置树或策略参数来使用前缀树。 请注意，这些参数现在已被弃用，并将在以后的版本中删除。

## 3. 重要提示

新的索引方法尚不支持以下功能：

- 使用MultiPoint几何类型的geo_shape查询-Elasticsearch当前禁止搜索具有MultiPoint几何类型的geo_shape字段，以避免在每个点上进行强力线性搜索。 现在，如果绝对需要这样做，可以使用每个点的布尔查询来实现。
- CONTAINS关系查询-使用新的默认矢量索引策略时，使用ElasticSearch 7.5.0或更高版本创建的索引支持将geo_shape查询定义为包含。



## 4.前缀树
[6.6]在6.6中弃用。不再使用PrefixTrees为了有效地表示倒排索引中的形状，使用PrefixTree的实现将Shapes转换为代表网格正方形的一系列哈希（通常称为“栅格”）。树的概念来自这样一个事实，即PrefixTree使用多个网格层，每个网格层的精度更高，以表示地球。可以认为这是在更高的缩放级别下增加地图或图像的细节级别。由于此方法会导致索引形状的精度问题，因此不建议使用矢量索引方法，该方法将形状索引为三角形网格（请参见索引方法）。

提供了多个PrefixTree实现：

- GeohashPrefixTree-对网格正方形使用geohash。 Geohash是交错的纬度和经度的位的base32编码的字符串。因此，哈希越长，它越精确。添加到geohash的每个字符代表另一个树级别，并为geohash增加5位精度。 Geohash表示一个矩形区域，并具有32个子矩形。 Elasticsearch中的最大级别数为24；默认值为9。
  
- QuadPrefixTree-将四叉树用于网格正方形。与geohash相似，四叉树将纬度和经度的比特交织在一起，结果哈希被置位。四叉树中的树级别代表此位集中的2位，每个坐标一个。 Elasticsearch中四叉树的最大级别数为29；默认值为21。



## 5.空间策略

[6.6]在6.6中弃用。 不再使用PrefixTrees选定的索引实现依赖于SpatialStrategy来选择如何分解形状（网格正方形或棋盘格三角网格）。 每种策略都回答以下问题：

- 可以索引哪种类型的形状？

- 可以使用哪些类型的查询操作和形状？

- 每个字段是否支持多个Shape？

  

提供以下策略实施（具有相应的功能）：

| 策略        | 支持的形状·                                                  | 支持的查询                                     | 多形状 |
| ----------- | ------------------------------------------------------------ | ---------------------------------------------- | ------ |
| `recursive` | [All](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/geo-shape.html#input-structure) | `INTERSECTS`, `DISJOINT`, `WITHIN`, `CONTAINS` | Yes    |
| `term`      | [Points](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/shape.html#point) | `INTERSECTS`                                   | Yes    |

## 6.准确性

​		递归和术语策略不提供100％的准确性，并且取决于它们的配置方式，它可能会为INTERSECTS，WITHIN和CONTAINS查询返回一些误报，而对于DISJOINT查询则返回一些误报。 为了减轻这种情况，为tree_levels参数选择适当的值并相应地调整期望很重要。 例如，一个点可能在特定网格像元的边界附近，因此可能与仅与它紧邻的像元匹配的查询不匹配-即使形状非常接近该点。



## 7.例子

```json
PUT /example
{
    "mappings": {
        "properties": {
            "location": {
                "type": "geo_shape"
            }
        }
    }
}
```

此映射定义使用默认矢量实现将location字段映射到geo_shape类型。 它提供大约1e-7的十进制度精度。



## 8.**前缀树的性能注意事项**

[6.6]在6.6中弃用。 PrefixTrees不再与前缀树一起使用，Elasticsearch将树中的路径用作倒排索引和查询中的术语。级别越高（精度越高），生成的项越多。当然，计算条件，将其保留在内存中以及将其存储在磁盘上都是有代价的。特别是对于较高的树级别，即使数据量适中，索引也可能变得非常大。此外，功能的大小也很重要。大而复杂的多边形在较高的树级别上会占用大量空间。哪种设置正确取决于使用情况。通常，要权衡准确性与索引大小和查询性能。

对于这两种实现，Elasticsearch的默认设置都是在索引大小和赤道处50m的合理精度水平之间做出折衷。这允许索引数千万个形状，而不会相对于输入大小过度膨胀最终的索引。



## 9. **输入结构**
可以使用GeoJSON或Well-Known Text（WKT）格式表示形状。 下表提供了GeoJSON和WKT到Elasticsearch类型的映射：

| GeoJSON Type         | WKT Type             | Elasticsearch Type   | Description                                                  |
| -------------------- | -------------------- | -------------------- | ------------------------------------------------------------ |
| `Point`              | `POINT`              | `point`              | 单个地理坐标。 注意：Elasticsearch仅使用WGS-84坐标。         |
| `LineString`         | `LINESTRING`         | `linestring`         | 给定两个或两个以上点的任意线。                               |
| `Polygon`            | `POLYGON`            | `polygon`            | 一个*封闭的*多边形，其第一个点和最后一个点必须匹配，因此需要'n + 1'个顶点才能创建一个'n'面的多边形，并且至少要有4个顶点。 |
| `MultiPoint`         | `MULTIPOINT`         | `multipoint`         | 一组未连接但可能相关的点。                                   |
| `MultiLineString`    | `MULTILINESTRING`    | `multilinestring`    | 单独的线串数组。                                             |
| `MultiPolygon`       | `MULTIPOLYGON`       | `multipolygon`       | 一组单独的多边形。                                           |
| `GeometryCollection` | `GEOMETRYCOLLECTION` | `geometrycollection` | 类似于`multi *`形状的GeoJSON形状，不同之处在于可以同时存在多种类型（例如Point和LineString）。 |
| `N/A`                | `BBOX`               | `envelope`           | 通过仅指定左上和右下点指定的边界矩形或信封。                 |
| `N/A`                | `N/A`                | `circle`             | 由中心点和半径（单位）指定的圆，默认为“ METERS”。            |



> 注意：对于所有类型，内部类型和坐标字段都是必需的。
>
> 在GeoJSON和WKT中，因此在Elasticsearch中，正确的坐标顺序是坐标数组中的经度，纬度（X，Y）。 这不同于许多通常使用口语经度（Y，X）的地理空间API（例如Google Maps）。





### 9.1 Point

点是单个地理坐标，例如建筑物的位置或智能手机的Geolocation API给出的当前位置。 以下是GeoJSON中一个点的示例。



```json
POST /example/_doc
{
    "location" : {
        "type" : "point",
        "coordinates" : [-77.03653, 38.897676]
    }
}
```

以下是WKT中一个点的示例：

```json
POST /example/_doc
{
    "location" : "POINT (-77.03653 38.897676)"
}
```



### 9.2 LineString

由两个或多个位置的数组定义的线串。 通过仅指定两个点，线串将代表一条直线。 指定两个以上的点会创建一条任意路径。 以下是GeoJSON中LineString的示例。

```json
POST /example/_doc
{
    "location" : {
        "type" : "linestring",
        "coordinates" : [[-77.03653, 38.897676], [-77.009051, 38.889939]]
    }
}
```

以下是WKT中一个LineString的示例：

```json
POST /example/_doc
{
    "location" : "LINESTRING (-77.03653 38.897676, -77.009051 38.889939)"
}
```

上面的线串将从白宫开始绘制一条直线，直到美国国会大厦。



### 9.3 多边形

多边形由点列表构成。 每个（外部）列表中的第一个点和最后一个点必须相同（多边形必须闭合）。 以下是GeoJSON中多边形的示例。

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ]
        ]
    }
}
```

以下是WKT中一个多边形的示例：

```json
POST /example/_doc
{
    "location" : "POLYGON ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0))"
}
```



第一个数组代表多边形的外部边界，其他数组代表内部形状（“孔”）。 以下是带有孔的多边形的GeoJSON示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ],
            [ [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2] ]
        ]
    }
}
```

以下是在WKT中带有孔的多边形的示例：

```json
POST /example/_doc
{
    "location" : "POLYGON ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8, 100.2 0.2))"
}
```

> 重要提示：WKT不会对顶点强制执行特定顺序，因此日期线和极点周围的多义多边形是可能的。 GeoJSON要求外部多边形必须为逆时针方向，内部形状必须为顺时针方向，这与用于顶点排序的开放地理空间联盟（OGC）简单要素访问规范一致。



如果Elasticsearch看起来不与dateline交叉（即，它们相交的经度小于180°），则它同时接受顺时针和逆时针的多边形，但是对于确实与dateline交叉的多边形（或对于宽于180°的其他多边形），Elasticsearch要求顶点排序 符合OGC和GeoJSON规范。 否则，可能会创建意外的多边形，并且将返回意外的查询/过滤结果。



以下提供了不明确多边形的示例。 Elasticsearch将应用GeoJSON标准来消除歧义，从而导致多边形越过dateline。

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [-177.0, 10.0], [176.0, 15.0], [172.0, 0.0], [176.0, -15.0], [-177.0, -10.0], [-177.0, 10.0] ],
            [ [178.2, 8.2], [-178.8, 8.2], [-180.8, -8.8], [178.2, 8.8] ]
        ]
    }
}
```



设置geo_shape映射时，可以定义orientation参数（请参见“映射选项”）。 这将为映射的geo_shape字段上的坐标列表定义顶点顺序。 也可以在每个文档上覆盖它。 以下是重写文档方向的示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "orientation" : "clockwise",
        "coordinates" : [
            [ [100.0, 0.0], [100.0, 1.0], [101.0, 1.0], [101.0, 0.0], [100.0, 0.0] ]
        ]
    }
}
```



### 9.4 MultiPoint多点

下面给出Geojson 多点的例子

```json
POST /example/_doc
{
    "location" : {
        "type" : "multipoint",
        "coordinates" : [
            [102.0, 2.0], [103.0, 2.0]
        ]
    }
}
```

下面给出WKT 多点的例子

```json
POST /example/_doc
{
    "location" : "MULTIPOINT (102.0 2.0, 103.0 2.0)"
}
```

### 9.5 [MultiLineString](http://www.geojson.org/geojson-spec.html#id6)

以下是geojson线串列表的示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "multilinestring",
        "coordinates" : [
            [ [102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0] ],
            [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0] ],
            [ [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8] ]
        ]
    }
}
```



以下是WKT线串列表的示例：

```json
POST /example/_doc
{
    "location" : "MULTILINESTRING ((102.0 2.0, 103.0 2.0, 103.0 3.0, 102.0 3.0), (100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8))"
}
```



### 9.6 MultiPolygon

以下是geojson多边形列表的示例（第二个多边形包含一个孔）：

```json
POST /example/_doc
{
    "location" : {
        "type" : "multipolygon",
        "coordinates" : [
            [ [[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]] ],
            [ [[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]],
              [[100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2]] ]
        ]
    }
}
```

以下是WKT多边形列表的示例（第二个多边形包含一个孔）：

```json
POST /example/_doc
{
    "location" : "MULTIPOLYGON (((102.0 2.0, 103.0 2.0, 103.0 3.0, 102.0 3.0, 102.0 2.0)), ((100.0 0.0, 101.0 0.0, 101.0 1.0, 100.0 1.0, 100.0 0.0), (100.2 0.2, 100.8 0.2, 100.8 0.8, 100.2 0.8, 100.2 0.2)))"
}
```



### 9.7 几何集合
以下是geojson几何对象集合的示例：

```json
POST /example/_doc
{
    "location" : {
        "type": "geometrycollection",
        "geometries": [
            {
                "type": "point",
                "coordinates": [100.0, 0.0]
            },
            {
                "type": "linestring",
                "coordinates": [ [101.0, 0.0], [102.0, 1.0] ]
            }
        ]
    }
}
```

以下是WKT几何对象集合的示例：

```json
POST /example/_doc
{
    "location" : "GEOMETRYCOLLECTION (POINT (100.0 0.0), LINESTRING (101.0 0.0, 102.0 1.0))"
}
```



### 9.8 Envelope

Elasticsearch支持包络类型，该包络类型由形状的左上角和右下角的坐标组成，以[[minLon，maxLat]，[maxLon，minLat]]格式表示边界矩形：

```json
POST /example/_doc
{
    "location" : {
        "type" : "envelope",
        "coordinates" : [ [100.0, 1.0], [101.0, 0.0] ]
    }
}
```

以下是使用WKT BBOX格式的信封的示例：

注意：WKT规范要求以下顺序：minLon，maxLon，maxLat，minLat。



### 9.9 圆
Elasticsearch支持一种圆形类型，该类型由具有半径的中心点组成。 请注意，只有在使用递归前缀树策略时，才能索引此圆表示形式。 对于默认的[Indexing approach](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/geo-shape.html#geoshape-indexing-approach)圆，应使用POLYGON近似圆。

```json
POST /example/_doc
{
    "location" : {
        "type" : "circle",
        "coordinates" : [101.0, 1.0],
        "radius" : "100m"
    }
}
```

注意：内半径字段为必填项。 如果未指定，则半径单位将默认为METERS。

注意：GeoJSON或WKT都不支持点半径圆类型。



## 10. 排序和检索索引形状

​		由于复杂的输入结构和形状的索引表示，当前无法对形状进行排序或直接检索其字段。 geo_shape值只能通过_source字段检索。




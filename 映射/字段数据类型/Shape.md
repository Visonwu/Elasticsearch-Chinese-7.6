# Shape 数据类型

shape数据类型有助于索引和搜索任意x，y笛卡尔形状，例如矩形和多边形。 它可用于索引和查询其坐标属于二维平面坐标系的几何。

您可以使用[shape Query](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-shape-query.html).此类型查询文档。





##  1.映射选项
像[geo_shape](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/geo-shape.html) 字段类型一样，形状字段映射将[GeoJSON](http://www.geojson.org/) or [Well-Known Text](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html)（WKT）几何对象映射到该形状类型。 要启用它，用户必须将字段显式映射到形状类型。

| 选项               | 描述                                                         | 默认值  |
| ------------------ | ------------------------------------------------------------ | ------- |
| `orientation`      | （可选）定义如何解释多边形/多多边形的顶点顺序。 此参数定义两个坐标系规则（右手或左手）之一，可以用三种不同的方式来指定每个规则。 1.右手规则：`right`，`ccw`，`逆时针`，2。左手规则：`left`，`cw`，`顺时针`。 默认方向（“逆时针方向”）符合OGC标准，该标准定义了逆时针顺序的外环顶点和顺时针顺序的内环顶点（孔）。 在geo_shape映射中设置此参数会显式设置geo_shape字段的坐标列表的顶点顺序，但可以在每个单独的GeoJSON或WKT文档中覆盖。 | `ccw`   |
| `ignore_malformed` | 如果为true，则会忽略格式错误的GeoJSON或WKT形状。 如果为假（默认），则格式错误的GeoJSON和WKT形状会引发异常并拒绝整个文档。 | `false` |
| `ignore_z_value`   | 如果为“ true”（默认），则将接受三个维度点（存储在源代码中），但仅会索引纬度和经度值； 第三维被忽略。 如果为false，则包含除纬度和经度（二维）值以外的值的地理点将引发异常并拒绝整个文档。 | `true`  |
| `coerce`           | 如果为true，则多边形中未闭合的线性环将自动闭合。             | `false` |



## 2.索引方法
像geo_shape一样，通过将几何分解为三角形网格并将每个三角形作为BKD树中的7维点进行索引，来索引形状字段类型。 提供给索引器的坐标是单精度浮点值，因此该字段保证了Java虚拟机提供的精度（通常为1E-38）。 对于多边形/多面体，镶嵌细分的性能主要取决于定义几何的顶点数量。



**重要笔记**

尚不支持以下功能：

- 具有MultiPoint几何类型的形状查询-Elasticsearch当前禁止搜索具有MultiPoint几何类型的形状字段，以避免对每个单个点进行强力线性搜索。 现在，如果绝对需要这样做，可以使用每个点的布尔查询来实现。 （注意：这可能会非常昂贵）
- CONTAINS关系查询-使用ElasticSearch 7.5.0或更高版本创建的索引支持定义为包含的关系的形状查询。



## 3. 例子

```json
PUT /example
{
    "mappings": {
        "properties": {
            "geometry": {
                "type": "shape"
            }
        }
    }
}
```

此映射定义将几何字段映射到形状类型。 索引器对顶点值使用单精度浮点数，因此可以保证精度与Java虚拟机提供的浮点数的精度大致相同（通常为1E-38）。



## 4.输入结构
可以使用 [GeoJSON](http://www.geojson.org/) or [Well-Known Text](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html) (WKT)格式表示形状。 下表提供了GeoJSON和WKT到Elasticsearch类型的映射：

| GeoJSON Type         | WKT Type             | Elasticsearch Type   | 描述                                                         |
| -------------------- | -------------------- | -------------------- | ------------------------------------------------------------ |
| `Point`              | `POINT`              | `point`              | 单一的x，y坐标。                                             |
| `LineString`         | `LINESTRING`         | `linestring`         | 给定两个或两个以上点的任意线。                               |
| `Polygon`            | `POLYGON`            | `polygon`            | 一个*封闭的*多边形，其第一个点和最后一个点必须匹配，因此需要'n + 1'个顶点才能创建一个'n'面的多边形，并且至少要有4个顶点。 |
| `MultiPoint`         | `MULTIPOINT`         | `multipoint`         | 一组未连接但可能相关的点。                                   |
| `MultiLineString`    | `MULTILINESTRING`    | `multilinestring`    | 单独的线串的数组。                                           |
| `MultiPolygon`       | `MULTIPOLYGON`       | `multipolygon`       | 一组单独的多边形。                                           |
| `GeometryCollection` | `GEOMETRYCOLLECTION` | `geometrycollection` | 形状集合类似于`multi *`形状，不同之处在于可以同时存在多种类型（例如Point和LineString）。 |
| `N/A`                | `BBOX`               | `envelope`           | 通过仅指定左上和右下点指定的边界矩形或信封。                 |



> 注：对于所有类型，内部类型和坐标字段都是必需的。
>
> 在GeoJSON和WKT中，因此在Elasticsearch中，正确的坐标顺序在坐标数组中为（X，Y）。 这与通常使用口语纬度，经度（Y，X）排序的许多Geospatial API（例如geo_shape）不同。



### 4.1 点
点是笛卡尔x，y空间中的单个坐标。 它可以表示感兴趣的项目在虚拟世界或投影空间中的位置。 以下是GeoJSON中一个点的示例

```json
POST /example/_doc
{
    "location" : {
        "type" : "point",
        "coordinates" : [-377.03653, 389.897676]
    }
}
```

以下是WKT中一个点的示例：

```json
POST /example/_doc
{
    "location" : "POINT (-377.03653 389.897676)"
}
```



### 4.2 LineString

​		由两个或多个位置的数组定义的线串。 通过仅指定两个点，线串将代表一条直线。 指定两个以上的点会创建一条任意路径。 以下是GeoJSON中LineLine的示例。 

```json
POST /example/_doc
{
    "location" : {
        "type" : "linestring",
        "coordinates" : [[-377.03653, 389.897676], [-377.009051, 389.889939]]
    }
}
```



以下是WKT中LineString的示例：

```json
POST /example/_doc
{
    "location" : "LINESTRING (-377.03653 389.897676, -377.009051 389.889939)"
}
```

### 4.3 多边形
多边形由点列表构成。 每个（外部）列表中的第一个点和最后一个点必须相同（多边形必须闭合）。 以下是GeoJSON中多边形的示例。

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [1000.0, -1001.0], [1001.0, -1001.0], [1001.0, -1000.0], [1000.0, -1000.0], [1000.0, -1001.0] ]
        ]
    }
}
```



以下是WKT中多边形的示例：

```json
POST /example/_doc
{
    "location" : "POLYGON ((1000.0 -1001.0, 1001.0 -1001.0, 1001.0 -1000.0, 1000.0 -1000.0, 1000.0 -1001.0))"
}
```

第一个数组代表多边形的外部边界，其他数组代表内部形状（“孔”）。 以下是带有孔的多边形的GeoJSON示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [1000.0, -1001.0], [1001.0, -1001.0], [1001.0, -1000.0], [1000.0, -1000.0], [1000.0, -1001.0] ],
            [ [1000.2, -1001.2], [1000.8, -1001.2], [1000.8, -1001.8], [1000.2, -1001.8], [1000.2, -1001.2] ]
        ]
    }
}
```

以下是在WKT中带有孔的多边形的示例：

```json
POST /example/_doc
{
    "location" : "POLYGON ((1000.0 1000.0, 1001.0 1000.0, 1001.0 1001.0, 1000.0 1001.0, 1000.0 1000.0), (1000.2 1000.2, 1000.8 1000.2, 1000.8 1000.8, 1000.2 1000.8, 1000.2 1000.2))"
}
```

> 注：重要说明：WKT不会对顶点强制执行特定顺序。 GeoJSON要求外部多边形必须为逆时针方向，内部形状必须为顺时针方向，这与用于顶点排序的开放地理空间联盟（OGC）简单要素访问规范一致。



默认情况下，Elasticsearch期望顶点按逆时针（右手规则）顺序。 如果按顺时针顺序提供数据（左手规则），则用户可以在字段映射中或作为文档随附的参数来更改方向参数。

以下是在文档上覆盖方向参数的示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "polygon",
        "orientation" : "clockwise",
        "coordinates" : [
            [ [1000.0, 1000.0], [1000.0, 1001.0], [1001.0, 1001.0], [1001.0, 1000.0], [1000.0, 1000.0] ]
        ]
    }
}
```



### 4.4 多点
以下是geojson点列表的示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "multipoint",
        "coordinates" : [
            [1002.0, 1002.0], [1003.0, 2000.0]
        ]
    }
}
```

以下是WKT点列表的示例：

```json
POST /example/_doc
{
    "location" : "MULTIPOINT (1002.0 2000.0, 1003.0 2000.0)"
}
```



### 4.5 MultiLineString

​	以下是geojson线串列表的示例：

```json
POST /example/_doc
{
    "location" : {
        "type" : "multilinestring",
        "coordinates" : [
            [ [1002.0, 200.0], [1003.0, 200.0], [1003.0, 300.0], [1002.0, 300.0] ],
            [ [1000.0, 100.0], [1001.0, 100.0], [1001.0, 100.0], [1000.0, 100.0] ],
            [ [1000.2, 100.2], [1000.8, 100.2], [1000.8, 100.8], [1000.2, 100.8] ]
        ]
    }
}
```

以下是WKT线串列表的示例：

```json
POST /example/_doc
{
    "location" : "MULTILINESTRING ((1002.0 200.0, 1003.0 200.0, 1003.0 300.0, 1002.0 300.0), (1000.0 100.0, 1001.0 100.0, 1001.0 100.0, 1000.0 100.0), (1000.2 0.2, 1000.8 100.2, 1000.8 100.8, 1000.2 100.8))"
}
```



### 4.6 多多边形
以下是geojson多边形列表的示例（第二个多边形包含一个孔）：

```json
POST /example/_doc
{
    "location" : {
        "type" : "multipolygon",
        "coordinates" : [
            [ [[1002.0, 200.0], [1003.0, 200.0], [1003.0, 300.0], [1002.0, 300.0], [1002.0, 200.0]] ],
            [ [[1000.0, 200.0], [1001.0, 100.0], [1001.0, 100.0], [1000.0, 100.0], [1000.0, 100.0]],
              [[1000.2, 200.2], [1000.8, 100.2], [1000.8, 100.8], [1000.2, 100.8], [1000.2, 100.2]] ]
        ]
    }
}
```

以下是WKT多边形列表的示例（第二个多边形包含一个孔）：

```json
POST /example/_doc
{
    "location" : "MULTIPOLYGON (((1002.0 200.0, 1003.0 200.0, 1003.0 300.0, 1002.0 300.0, 102.0 200.0)), ((1000.0 100.0, 1001.0 100.0, 1001.0 100.0, 1000.0 100.0, 1000.0 100.0), (1000.2 100.2, 1000.8 100.2, 1000.8 100.8, 1000.2 100.8, 1000.2 100.2)))"
}
```



### 4.7 几何集合
以下是geojson几何对象集合的示例：

```json
POST /example/_doc
{
    "location" : {
        "type": "geometrycollection",
        "geometries": [
            {
                "type": "point",
                "coordinates": [1000.0, 100.0]
            },
            {
                "type": "linestring",
                "coordinates": [ [1001.0, 100.0], [1002.0, 100.0] ]
            }
        ]
    }
}
```

以下是WKT几何对象集合的示例：

```json
POST /example/_doc
{
    "location" : "GEOMETRYCOLLECTION (POINT (1000.0 100.0), LINESTRING (1001.0 100.0, 1002.0 100.0))"
}
```



### 4.8 Envelope

信封Elasticsearch支持Envelope类型，该Envelope类型由形状的左上角和右下角的坐标组成，以[[minX，maxY]，[maxX，minY]]格式表示边界矩形：

```json
POST /example/_doc
{
    "location" : {
        "type" : "envelope",
        "coordinates" : [ [1000.0, 100.0], [1001.0, 100.0] ]
    }
}
```

以下是使用WKT BBOX格式的Envelope的示例：

注意：WKT规范要求以下顺序：minLon，maxLon，maxLat，minLat。

```json
POST /example/_doc
{
    "location" : "BBOX (1000.0, 1002.0, 2000.0, 1000.0)"
}
```



## 5.排序和检索索引形状
由于复杂的输入结构和形状的索引表示，当前无法对形状进行排序或直接检索其字段。 形状值只能通过_source字段检索。








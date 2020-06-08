# format

在JSON文档中，日期以字符串表示。 Elasticsearch使用一组预配置的格式来识别这些字符串并将其解析为一个长值，该长值表示UTC以来的毫秒数。

除了内置格式外，还可以使用熟悉的yyyy / MM / dd语法指定自己的自定义格式：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "date": {
        "type":   "date",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

许多支持日期值的API也支持 [date math](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/common-options.html#date-math)表达式，例如`now-1m / d`-当前时间（减去一个月），四舍五入到最接近的日期。



## 1.自定义日期格式
支持完全可自定义的日期格式。 这些语法在DateTimeFormatter文档中进行了说明。



## 2.内置格式
下面的大多数格式都具有严格的伴随格式，这意味着一周中的年，月和日部分必须分别准确地使用4位，2位和2位数字，并可能以零开头。 例如，像5/11/1这样的日期将被认为是无效的，需要将其重写为2005/11/01才能被日期解析器接受。



要使用它们，您需要在strict_之前加上日期格式的名称，例如strict_date_optional_time而不是date_optional_time。

当 [date fields are dynamically mapped](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/dynamic-field-mapping.html#date-detection)字段以确保不会意外地将无关的字符串映射为日期时，这些严格的日期格式特别有用。

下表列出了所有支持的默认ISO格式：

**`epoch_millis`**： 一种格式表示自该纪元以来的毫秒数。 请注意，此时间戳受Java Long.MIN_VALUE和Long.MAX_VALUE的限制。



**`epoch_second`**：自该时期以来的秒数的格式程序。 请注意，此时间戳受Java“ Long.MIN_VALUE”和“ Long”的限制。 MAX_VALUE`除以1000（毫秒数）。



**`date_optional_time` or `strict_date_optional_time`**：一个通用的ISO日期时间解析器，其中必须使用“ year_month_day”格式的日期，而由“ T”分隔的时间是可选的。 例如：`yyyy-MM-dd'T'HH：mm：ss.SSSZ`或`yyyy-MM-dd`。



**`basic_date`**：完整日期的基本格式，如四位数的年份，两位数的月份和月份的两位数的日期：`yyyyMMdd`。



**`basic_date_time`**：一个基本的格式化程序，它结合了一个基本的日期和时间，并用* T *分隔：yyyyMMdd'T'HHmmss.SSSZ。



**`basic_date_time_no_millis`**：基本格式器，结合了基本日期和时间（不含毫秒），并以* T *分隔：yyyyMMdd'T'HHmmssZ。



**`basic_ordinal_date`**：格式为完整序号日期的格式器，使用四位数的年份和三位数的dayOfYear：yyyyDDD。



**`basic_ordinal_date_time`**：格式为完整序号日期和时间的格式程序，使用四位数的年份和三位数的dayOfYear：“ yyyyDDD'T'HHmmss.SSSZ”。



**`basic_ordinal_date_time_no_millis`**

一个格式化的格式，用于使用四位数的年份和三位数的dayOfYear来表示无毫秒的完整序数日期和时间：yyyyDDD'T'HHmmssZ。

**`basic_time`**：一个基本的格式化程序，用于表示一天的两位小时，两位的小时分钟，两位的秒分钟，三位的毫秒以及时区偏移量：“ HHmmss.SSSZ”。



**`basic_time_no_millis`**：每天两位数小时，两位小时数分钟，两位秒分钟数和时区偏移量的基本格式：“ HH mm ssZ”。



**`basic_t_time`**：一个基本的格式化程序，用于表示一天的两位小时，两位的小时分钟，两位的秒分钟，三位的毫秒以及以* T *为前缀的时区偏移量：''T'HHmmss.SSSZ'。



**`basic_t_time_no_millis`**：一个基本的格式化程序，用于表示一天的两位小时，两位的小时分钟，两位的秒分钟以及以* T *为前缀的时区偏移量：“'T'HHmmssZ”。

​	

**`basic_week_date` or `strict_basic_week_date`**：基本格式，用于将完整日期设置为四位数的工作日周，两位数字的工作日周和一位数字的星期几：`xxxx'W'wwe。



**`basic_week_date_time` or `strict_basic_week_date_time：一个基本的格式化程序，结合了基本的星期日期和时间，并以* T *分隔：`xxxx'W'wwe'T'HHmmss.SSSZ`。



**`basic_week_date_time_no_millis` or `strict_basic_week_date_time_no_millis`**：一个基本的格式化程序，它结合了基本的星期日期和时间（不含毫秒），并以* T *分隔：`xxxx'W'wwe'T'HHmmssZ`。



**`date` or `strict_date`**：格式化日期的完整日期，如四位数的年份，两位数的月份和月份的两位数的日期：`yyyy-MM-dd`。



**`date_hour` or `strict_date_hour`**：结合了完整日期和一天中两位数小时的格式化程序：“ yyyy-MM-dd'T'HH”。



**`date_hour_minute` or `strict_date_hour_minute`**：格式化程序，它结合了完整的日期，一天中的两位数小时和小时中的两位数分钟：“ yyyy-MM-dd'T'HH：mm”。



**`date_hour_minute_second` or `strict_date_hour_minute_second`**：格式化程序，它结合了完整的日期，一天中的两位数小时，小时的两位数分钟和分钟的两位数秒：“ yyyy-MM-dd'T'HH：mm：ss”。



**`date_hour_minute_second_fraction` or `strict_date_hour_minute_second_fraction`**：格式化程序，它结合了完整的日期，一天中的两位数小时，小时的两位数分钟，分钟的两位数秒和三分之一秒：`yyyy-MM-dd'T'HH：mm：ss.SSS` 。



**`date_hour_minute_second_millis` or `strict_date_hour_minute_second_millis`**：格式化程序，它结合了完整的日期，一天中的两位数小时，小时的两位数分钟，分钟的两位数秒和三分之一秒：`yyyy-MM-dd'T'HH：mm：ss.SSS` 。



**`date_time` or `strict_date_time`**：结合了完整日期和时间的格式化程序，以* T *：`yyyy-MM-dd'T'HH：mm：ss.SSSZZ`分隔。



**`date_time_no_millis` or `strict_date_time_no_millis`**：结合了完整日期和时间的格式化程序，以* T *：`yyyy-MM-dd'T'HH：mm：ss.SSSZZ`分隔。...



**`hour` or `strict_hour`**`：一天中两位数小时的格式化程序：`HH



**`hour_minute` or `strict_hour_minute`**：格式化时间为两位数的小时和两位数的小时：“ HH：mm”。



**`hour_minute_second` or `strict_hour_minute_second`**：格式化时间为两位数的小时和两位数的小时：“ HH：mm”。...



**`hour_minute_second_fraction` or `strict_hour_minute_second_fraction`**：格式，用于表示一天的两位数小时，两位的小时数分钟，两位的分钟数秒和三位数的秒数：“ HH：mm：ss.SSS”。



**`hour_minute_second_millis` or `strict_hour_minute_second_millis`**：格式，用于表示一天的两位数小时，两位的小时数分钟，两位的分钟数秒和三位数的秒数：“ HH：mm：ss.SSS”。



**`ordinal_date` or `strict_ordinal_date`**：格式完整的日期，使用四位数的年份和三位数的dayOfYear：“ yyyy-DDD”。



**`ordinal_date_time` or `strict_ordinal_date_time`**：格式为完整序号日期和时间的格式程序，使用四位数的年份和三位数的dayOfYear：“ yyyy-DDD'T'HH：mm：ss.SSSZZ”。



**`ordinal_date_time_no_millis` or `strict_ordinal_date_time_no_millis`**：格式化程序，用于使用四位数的年份和三位数的dayOfYear来表示不带毫秒的完整序数日期和时间：yyyy-DDD'T'HH：mm：ssZZ。



**`time` or `strict_time`**：格式，用于表示一天的两位小时，两位的小时分钟，两位的秒分钟，三位数的秒和时区偏移量：“ HH：mm：ss.SSSZZ”。



**`time_no_millis` or `strict_time_no_millis`**：格式，用于表示一天的两位小时，两位的小时分钟，两位的秒分钟和时区偏移量：“ HH：mm：ssZZ”。



**`t_time` or `strict_t_time`**：格式为一天中的两位小时，每小时的两位分钟，分钟的两位秒，秒的三位数和时区偏移量的前缀* T *：`'T HH：mm：ss.SSSZ`。



**`t_time_no_millis` or `strict_t_time_no_millis`**：格式为两位小时的小时，两位小时的分钟，两位秒的分钟以及以* T *：'T HH：mm：ssZ'为前缀的时区偏移量。



**`week_date` or `strict_week_date`**：格式化日期的完整日期，如四位数字的周年，两位数字的周年和一周的某天：`xxxx-'W'ww-e'。



**`week_date_time` or `strict_week_date_time`**：格式化程序，结合了整周的日期和时间，并用* T *：`xxxx-'W'ww-e'T'HH：mm：ss.SSSZZ`分隔。



**`week_date_time_no_millis` or `strict_week_date_time_no_millis`**：格式化程序，结合了整周的日期和时间（不含毫秒），并用* T *：`xxxx-'W'ww-e'T'HH：mm：ssZZ`分隔。



**`weekyear` or `strict_weekyear`**：每周四位数的格式化程序：`xxxx`。



**`weekyear_week` or `strict_weekyear_week`**：格式为年份的四位数和年份的两位数的年份：“ yyyy-MM”。



**`weekyear_week_day` or `strict_weekyear_week_day`**：四位数的年份，一年的两位数月份和月份的两位数日期的格式化程序：“ yyyy-MM-dd”。...



**`year` or `strict_year`**：四位数的年份，一年的两位数月份和月份的两位数日期的格式化程序：“ yyyy-MM-dd”。...



**`year_month` or `strict_year_month`**：四位数的年份，一年的两位数月份和月份的两位数日期的格式化程序：“ yyyy-MM-dd”。...



**`year_month_day` or `strict_year_month_day`**：四位数的年份，一年的两位数月份和月份的两位数日期的格式化程序：“ yyyy-MM-dd”。


















## Druid查询

## 一、 聚合查询
### 1. Timeseries
时间序列查询，不需要指定metric，可进行聚合和过滤
   
查询属性：

property| description| required？|
--|--|--|
 queryType|查询类型，必须是"timeseries" | yes|
dataSource |	数据源名称 |yes |
 intervals|查询数据的时间范围 |yes |
descending |是否排序，默认false为升序，true为降序 |no |
 filter|查询的过滤条件 |no |
granularity | 查询时间的粒度| yes|
aggregations | 聚合操作指标，按照metrics列进行统计|yes |
 postAggregations|在aggregations的基础上，进行一些再操作，支持+,-,*,/ 等 |no |

查询格式：

```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "sample_dimension1", "value": "sample_value1" },
      { "type": "or",
        "fields": [
          { "type": "selector", "dimension": "sample_dimension2", "value": "sample_value2" },
          { "type": "selector", "dimension": "sample_dimension3", "value": "sample_value3" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" },
    { "type": "doubleSum", "name": "sample_name2", "fieldName": "sample_fieldName2" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "name": "postAgg__sample_name1", "fieldName": "sample_name1" },
        { "type": "fieldAccess", "name": "postAgg__sample_name2", "fieldName": "sample_name2" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ]
}
```

响应格式：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> } 
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> }
  }
]
```

零填充问题：
        
若在查询的时间粒度中，某些segment的数据为空，则会用0来表示，默认采用此方式。比如，按查询某几天数据，按照day为单位，某一天数据为不存在，则会返回如下格式：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  },
  {
   "timestamp": "2012-01-02T00:00:00.000Z",
   "result": { "sample_name1": 0 }
  },
  {
    "timestamp": "2012-01-03T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  }
]
```

若不想出现为0的状况，则可以请求时，进行如下操作：

```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-04T00:00:00.000" ],
  "context" : {
    "skipEmptyBuckets": "true"
  }
}
```

### 2. TopN
TopN查询，是指定一个维度进行结果的排序统计。类似于groupBy单个维度，ordering spec排序统计条件。默认的topN是最大值1000，max(1000,threshold)。必须带有metric指标
    
查询属性：

 property| description| required？|
---|---|--|
 queryType| 查询类型，必须是select| yes|
 dataSource| 数据源名称|yes |
 intervals|查询数据的时间范围 |yes |
 granularity|定义bucket查询结果的粒度 | no|
 filter|查询的过滤条件 |no |
 dimensions|定义维度的string或者json对象，top操作需要统计的维度，相当于groupby中的单个维度 |yes |
 aggregations|聚合操作指标，按照metrics列进行统计 |no |
 postAggregations|在aggregations的基础上，进行一些再操作，支持+,-,*,/ 等 |no |
 threshold| topN的整数，指结果中会有多少条记录|no |
 metric|指定metric的string或者json对象 | yes|

查询格式：

```json
{
  "queryType": "topN",
  "dataSource": "sample_data",
  "dimension": "sample_dim",
  "threshold": 5,
  "metric": "count",
  "granularity": "all",
  "filter": {
    "type": "and",
    "fields": [
      {
        "type": "selector",
        "dimension": "dim1",
        "value": "some_value"
      },
      {
        "type": "selector",
        "dimension": "dim2",
        "value": "some_other_val"
      }
    ]
  },
  "aggregations": [
    {
      "type": "longSum",
      "name": "count",
      "fieldName": "count"
    },
    {
      "type": "doubleSum",
      "name": "some_metric",
      "fieldName": "some_metric"
    }
  ],
  "postAggregations": [
    {
      "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        {
          "type": "fieldAccess",
          "name": "some_metric",
          "fieldName": "some_metric"
        },
        {
          "type": "fieldAccess",
          "name": "count",
          "fieldName": "count"
        }
      ]
    }
  ],
  "intervals": [
    "2013-08-31T00:00:00.000/2013-09-03T00:00:00.000"
  ]
}
```

响应格式：

```json
[
  {
    "timestamp": "2013-08-31T00:00:00.000Z",
    "result": [
      {
        "dim1": "dim1_val",
        "count": 111,
        "some_metrics": 10669,
        "average": 96.11711711711712
      },
      {
        "dim1": "another_dim1_val",
        "count": 88,
        "some_metrics": 28344,
        "average": 322.09090909090907
      },
      {
        "dim1": "dim1_val3",
        "count": 70,
        "some_metrics": 871,
        "average": 12.442857142857143
      },
      {
        "dim1": "dim1_val4",
        "count": 62,
        "some_metrics": 815,
        "average": 13.14516129032258
      },
      {
        "dim1": "dim1_val5",
        "count": 60,
        "some_metrics": 2787,
        "average": 46.45
      }
    ]
  }
]
```


### 3. GroupBy

## 二、元数据信息查询
   ### 1. Time Boundary
 此查询可以查询出数据集中，数据最早的时间戳和最晚的时间戳。即存入库中的数据的最早和最晚时间。
    
查询属性：

 property| description|required? |
---|---|---
 queryType| 查询类型，必须是"timeBoundary" |  yes|
 dataSource| 数据源，相当于关系数据库里面的table	 | yes | 
 bound| 可选项，可以设置为maxTime（最新时间）或者minTime（最早时间），默认两个都返回 |no


查询格式：

```json
{
    "queryType" : "timeBoundary",
    "dataSource": "sample_datasource",
    "bound"     : < "maxTime" | "minTime" > # optional, defaults to returning both timestamps if not set 
}
```

响应格式：

```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "minTime" : "2013-05-09T18:24:00.000Z",
    "maxTime" : "2013-05-09T18:37:00.000Z"
  }
} ]
```


####    2. Segment Metadata
Segment查询会返回底层存储的每个segment信息，包括以下内容：
- 所有列的基数Cardinality
- 列类型为string的最大、最小值
- 列所占用的空间估值byte
- segment中存储的行数
- segment的时间区域
- 所有列的类型
- 存储的所有segment空间大小估值byte
- segment id
        查询属性：

property | description | required?
---|---|---
queryType | 查询类型，比如是"segmentMetadata" |yes
dataSource | 数据源名称 |yes
intervals | 时间范围|no
toInclude | 哪些列被返回，默认是"all"|no
merge | 把所有独立的segment元数据合并为一个结果集 |no
analysisTypes | 指定哪些列的属性(e.g. cardinality,size)被计算和返回，默认是["cardinality","size","interval","minmax"] |no
lenientAggregatorMerge | 若为true并且"aggregators" 分析类型是可用的，aggregators会很轻松被合并。 |no

查询格式：

```json
{
  "queryType":"segmentMetadata",
  "dataSource":"sample_datasource",
  "intervals":["2013-01-01/2014-01-01"]
}
```

响应格式：

```json
[ {
  "id" : "some_id",
  "intervals" : [ "2013-05-13T00:00:00.000Z/2013-05-14T00:00:00.000Z" ],
  "columns" : {
    "__time" : { "type" : "LONG", "hasMultipleValues" : false, "size" : 407240380, "cardinality" : null, "errorMessage" : null },
    "dim1" : { "type" : "STRING", "hasMultipleValues" : false, "size" : 100000, "cardinality" : 1944, "errorMessage" : null },
    "dim2" : { "type" : "STRING", "hasMultipleValues" : true, "size" : 100000, "cardinality" : 1504, "errorMessage" : null },
    "metric1" : { "type" : "FLOAT", "hasMultipleValues" : false, "size" : 100000, "cardinality" : null, "errorMessage" : null }
  },
  "aggregators" : {
    "metric1" : { "type" : "longSum", "name" : "metric1", "fieldName" : "metric1" }
  },
  "queryGranularity" : {
    "type": "none"
  },
  "size" : 300000,
  "numRows" : 5000000
} ]
```

toInclude的设置有下面三种方式：

```json
"toInclude":{"type":"all"}
"toInclude":{"type":"none"}
"toInclude":{"type":"list","columns":[<string list of column names>]}
```


#### 3. Datasource Metadata
Datasource查询只会返回所有数据源的最新一条数据记录的时间（time字段）。
 
 查询格式：

```json
{
    "queryType" : "dataSourceMetadata",
    "dataSource": "sample_datasource"
}
```

响应格式：

```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "maxIngestedEventTime" : "2013-05-09T18:24:09.007Z",
  }
} ]
```


## 三、Search查询
1. search
    
   search查询相当于查询在满足某种匹配查询条件下，统计在查询指定某些维度的count值。
    查询格式：

```json
{
  "queryType": "search",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "searchDimensions": [
    "dim1",
    "dim2"
  ],
  "query": {
    "type": "insensitive_contains",
    "value": "Ke"
  },
  "sort" : {
    "type": "lexicographic"
  },
  "intervals": [
    "2013-01-01T00:00:00.000/2013-01-03T00:00:00.000"
  ]
}
```

    响应格式：

```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": [
      {
        "dimension": "dim1",
        "value": "Ke$ha",
        "count": 3
      },
      {
        "dimension": "dim2",
        "value": "Ke$haForPresident",
        "count": 1
      }
    ]
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": [
      {
        "dimension": "dim1",
        "value": "SomethingThatContainsKe",
        "count": 1
      },
      {
        "dimension": "dim2",
        "value": "SomethingElseThatContainsKe",
        "count": 2
      }
    ]
  }
]
```

## 四、Select查询
select可以查询明细数据，即可以查询指定维度的数据，然后将其返回。
    
查询格式：

```json
{
   "queryType": "select",
   "dataSource": "wikipedia",
   "descending": "false",
   "dimensions":[],
   "metrics":[],
   "granularity": "all",
   "intervals": [
     "2013-01-01/2013-01-02"
   ],
   "pagingSpec":{"pagingIdentifiers": {}, "threshold":5}
 }
```

查询属性：

 property| description| required？|
 ---|---|---|
 queryType| 查询类型，必须是select| yes|
 dataSource| 数据源名称|yes |
 intervals|查询数据的时间范围 |yes |
 descending| 是否排序，默认false为升序，true表示降序|no |
 filter| 查询的过滤条件|no |
 dimensions| 维度列表，若为空则返回数据所有维度，否则返回指定维度数据|no |
 metrics| 指标列表，若为空则返回所有的指标信息，否则返回指定指标数据|no |
 pagingSpec|json对象，返回数据的页码偏移量，threshold返回的数据量数目 |yes |

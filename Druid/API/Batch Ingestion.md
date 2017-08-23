### Batch Ingestion

离线批处理摄入数据是通过Druid的Overload节点进行的。

任务提交格式：

```
http://OVERLORD_IP:8090/druid/indexer/v1/task
```

格式样例：

```
curl -X 'POST' -H 'Content-Type:application/json' -d @my-index-task.json OVERLORD_IP:8090/druid/indexer/v1/task
```
或者所有任务在本地运行，采用localhost方式：

```
curl -X 'POST' -H 'Content-Type:application/json' 10.21.xx.xxx:8090/druid/indexer/v1/task -d @hdmb-171-index.json
```

paths可以有多个，并且以逗号分隔，例如：

```
"paths":"/path1,/path2,/path3"
```
paths要摄取数据源路径，json格式的文本数据，或者从hadoop集群中获取数据源，那么相对应的路径需要改成hadoop的hdfs路径。

若type为index_hadoop表示从hadoop文件系统中获取数据源，因此paths的配置是hadoop中的路径

若type为index表示可以从本地获取文件，此时文件的路径由ioConfig中进行配置，baseDir为数据源的目录，filter为文件本身。例如

```json
"ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/indexing/",
        "filter" : "wikipedia_data.json"
       }
}
```


Batch Ingestion样例：
```json
{
    "type":"index_hadoop",
    "spec":{
        "ioConfig":{
            "type":"hadoop",
            "inputSpec":{
                "type":"static",
                "paths":"/data10/druid/source/hdmb_171_20170207.json"
            }
        },
        "dataSchema":{
            "dataSource":"hdmb_171_20170207",
            "granularitySpec":{
                "type":"uniform",
                "segmentGranularity":"hour",
                "queryGranularity":"none",
                "rollup": false,
                "intervals":["2017-02-07/2017-02-08"]
            },
            "parser":{
                "type":"string",
                "parseSpec":{
                      "format":"json",
                      "dimensionsSpec":{
                         "dimensions":["guid","key","pid","province","uid","sdkver","sr","sys","mbos","ver","from","net","ip","city","country","mbl","sjp","sjm","ntm","hdid","source"]
                    },
                      "timestampSpec":{
                         "format":"yyyy-MM-dd HH:mm",
                         "column":"date"
                    }
                }
            },
            "metricsSpec":[
               {
                  "name":"count","type":"count"
               },
               {
                  "name":"user_unique",
                  "type":"hyperUnique",
                  "fieldName":"key"
               }
            ]
        },
        "tuningConfig":{
            "type":"hadoop",
            "partitionsSpec":{
               "type":"hashed",
               "targetPartitionSize":5000000
            },
            "jobProperties":{}
        }
    }
    
}
```

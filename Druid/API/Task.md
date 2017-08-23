
### Task：

#### 数据摄入Task

##### Index Task
Index task的type类型必须是index。firehose中配置数据源的路径。

ioConfig中的配置如下：

```json
"ioConfig":{
    "type":"index",
    "firehose":{
        "type":"local",
        "baseDir":"/data10/druid/source/",
        "filter":"hdmb_171_20170207.json"
    }
}
```

tuningConfig中的配置如下：


```json
"tuningConfig":{
    "type":"index",
    "targetPartitionSize":5000000
}
```



==注：此数据源是本地机器路径地址（必须是MiddleManager节点启动的机器，否则会出现异常）==

##### Hadoop Index Task



#### Segment删除Task

kill Task

kill task会删除关于segment的所有信息并且会从deep storage中移除。


语法如下：
```json
{
    "type": "kill",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "interval" : <all_segments_in_this_interval_will_die!>
}
```

Eg：
```json
{
    "type": "kill",
    "id": "hdmb_index_only",
    "dataSource": "hdmb_171_20170207_index",
    "interval" : ["2017-02-07/2017-02-08"]
}
```

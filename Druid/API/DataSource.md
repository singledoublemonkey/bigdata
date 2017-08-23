
#### 单表查询——Table Data Source

有两种方式：

第一种：直接"dataSource":"index_name"

第二种：
```json
{
    "type": "table",
    "name": "<string_value>"
}
```

#### 多表联合查询——Union Data Source

联合2个多以上的多表进行查询

```json
{
    "type": "union",
    "dataSources": ["<string_value1>", "<string_value2>", "<string_value3>", ... ]
}
```

注：联表查询，这些数据源必须有相同的schema，联合查询的数据会发送到broker/router节点，不会经过History节点

#### Query Data Source

此方式仅支持嵌入group by操作

```json
{
    "type": "query",
    "query": {
        "type": "groupBy",
        ...
    }
}
```

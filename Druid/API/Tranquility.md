Tranquility允许使用自己的应用程序将解析封装的数据导入Druid中，例如JVM程序。
    
Tranquility包括好几个模块，可以自由选择，例如core、storm、spark、Server、Samza、Kafka、Flink
- Core —— 是基本的数据发送API，如果不是用类似storm等其他API接口，使用程序组装的数据格式，则可以使用此API。
- Storm —— Tranquility包括了一个storm bolt和一个trident state
- Spark —— Tranquility通过RDDS和DStreams工作
- Server —— HTTP服务允我们不用开发JVM应用程序情况下使用Tranquility
- Samza —— Tranquility包括一个Samza SystemProducer
- Kafka —— 应用程序通过Tranquility将Kafka的消息推送到Druid
- Flink —— Tranquility包括一个Flink Sink

通过Maven获取Tranquility
    

```xml
<dependency>
  <groupId>io.druid</groupId>
  <artifactId>tranquility-core_2.11</artifactId>
  <version>0.8.1</version>
</dependency>
<dependency>
  <groupId>io.druid</groupId>
  <artifactId>tranquility-samza_2.10</artifactId>
  <version>0.8.1</version>
</dependency>
<dependency>
  <groupId>io.druid</groupId>
  <artifactId>tranquility-spark_2.11</artifactId>
  <version>0.8.1</version>
</dependency>
<dependency>
  <groupId>io.druid</groupId>
  <artifactId>tranquility-storm_2.11</artifactId>
  <version>0.8.1</version>
</dependency>
<dependency>
  <groupId>io.druid</groupId>
  <artifactId>tranquility-flink_2.11</artifactId>
  <version>0.8.1</version>
</dependency>
```

我们可以根据自己的需求加入相应的依赖。


>Spark中flatMap和map的区别，reduce和reduceByKey的区别
## flatMap VS map：

### 1. map(func)

将原数据的每个元素传给函数func进行格式化，返回一个新的分布式数据集。(原文：Return a new distributed dataset formed by passing each element of the source through a function func.)

### 2. flatMap(func)

跟map(func)类似，但是每个输入项和成为0个或多个输出项(所以func函数应该返回的是一个序列化的数据而不是单个数据项)。(原文：Similar to map, but each input item can be mapped to 0 or more output items (so func should return a Seq rather than a single item).)

>在使用时map会将一个长度为N的RDD转换为另一个长度为N的RDD；而flatMap会将一个长度为N的RDD转换成一个N个元素的集合，然后再把这N个元素合成到一个单个RDD的结果集。

**Example:**

对于文本a.txt，假设此文本在本地文件目录/home/spark下
有如下内容
```
a c d e
c e
b
```
操作如下：

```scala
val text = sc.textFile("file:///home/spark/a.txt")
val rdd1 = text.map(line=>line.split("\\s+")) // equals:text.map(_.split("\\s+"))
val rdd2 = text.flatMap(line=>line.split("\\s+")) // equals: text.flatMap(_.split("\\s+"))

# output: Array[Array[String]] = Array(Array(a,c,d,e),Array(c,e),Array(b))
rdd1.collect()

# output: Array[String] = Array(a,c,d,e,c,e,b)
rdd2.collect()
```

flatMap做了两个步骤，第一步和map效果一样，每一行针对函数拆分成独立的RDD；第二步则是将第一步拆分的每个RDD进行合成。

## reduce VS reduceByKey

### 1. reduce
reduce将RDD中元素前两个值传给输入函数，产生一个新的return值，新产生的return值与RDD中下一个元素（第三个元素）组成两个元素，再被传给输入函数，直到最后只有一个值为止。

### 2. reduceByKey
reduceByKey是对元素为KV对的RDD中Key相同的元素的Value进行指定函数的reduce操作，因此，Key相同的多个元素的值被reduce为一个值，然后与原RDD中的Key组成一个新的KV对。

**Example：**

```scala
// reduce
val rdd = sc.parallelize(1 to 5)
rdd.collect()   //Array[Int] = Array(1, 2, 3, 4, 5)
rdd.reduce((a,b)=>a+b)  //ouput: 15

//reduceByKey
val text = sc.textFile("file:///home/spark/a.txt")
val wordcount = text.flatMap(line=>line.split("\\s+")).map(word=>(word,1)).reduceByKey(_+_)
wordcount.collect() // Array[(String, Int)] = Array((d,1), (b,1), (e,2), (a,1), (c,2))
```

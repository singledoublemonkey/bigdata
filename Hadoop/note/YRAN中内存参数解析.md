## Container

Container就是一个yarn的java进程，在Mapreduce中的AM，MapTask，ReduceTask都作为Container在Yarn的框架上执行，你可以在RM的网页上看到Container的状态

### 基础

Yarn的ResourceManger（简称RM）通过逻辑上的队列分配内存，CPU等资源给application，默认情况下RM允许最大AM申请Container资源为8192MB("yarn.scheduler.maximum-allocation-mb")，默认情况下的最小分配资源为1024M("yarn.scheduler.minimum-allocation-mb")，AM只能以增量（"yarn.scheduler.minimum-allocation-mb"）和不会超过("yarn.scheduler.maximum-allocation-mb")的值去向RM申请资源，AM负责将("mapreduce.map.memory.mb")和("mapreduce.reduce.memory.mb")的值规整到能被("yarn.scheduler.minimum-allocation-mb")整除，RM会拒绝申请内存超过8192MB和不能被1024MB整除的资源请求。

### 相关参数

**YARN**

- yarn.scheduler.minimum-allocation-mb
- yarn.scheduler.maximum-allocation-mb
- yarn.nodemanager.vmem-pmem-ratio
- yarn.nodemanager.resource.memory.mb

**MapReduce**

Map Memory

- mapreduce.map.java.opts
- mapreduce.map.memory.mb

Reduce Memory

- mapreduce.reduce.java.opts
- mapreduce.reduce.memory.mb

以map container内存分配("mapreduce.map.memory.mb")设置为1536为例，AM将会为container向RM请求2048mb的内存资源，因为最小分配单位("yarn.scheduler.minimum-allocation-mb")被设置为1024，这是一种逻辑上的分配，这个值被NodeManager用来监控改进程内存资源的使用率，如果map Task堆的使用率超过了2048MB，NM将会把这个task给杀掉，JVM进程堆的大小被设置为1024("mapreduce.map.java.opts=-Xmx1024m")适合在逻辑分配为2048MB中，同样reduce container("mapreduce.reduce.memory.mb")设置为3072也是.

当一个mapreduce job完成时，你将会看到一系列的计数器被打印出来，下面的三个计数器展示了多少物理内存和虚拟内存被分配

Physical memory (bytes) snapshot=21850116096
Virtual memory (bytes) snapshot=40047247360
Total committed heap usage (bytes)=22630105088

虚拟内存

默认的("yarn.nodemanager.vmem-pmem-ratio")设置为2.1，意味则map container或者reduce container分配的虚拟内存超过2.1倍的("mapreduce.reduce.memory.mb")或("mapreduce.map.memory.mb")就会被NM给KILL掉，如果 ("mapreduce.map.memory.mb") 被设置为1536那么总的虚拟内存为2.1*1536=3225.6MB

当container的内存超出要求的，log将会打印一下信息

```
Current usage: 2.1gb of 2.0gb physical memory used; 1.6gb of 3.15gb virtual memory used. Killing container.
```
**mapreduce.map.java.opts和mapreduce.map.memory.mb**

大概了解完以上的参数之后，mapreduce.map.java.opts和mapreduce.map.memory.mb参数之间，有什么联系呢？

通过上面的分析，我们知道如果一个yarn的container超除了heap设置的大小，这个task将会失败，我们可以根据哪种类型的container失败去相应增大mapreduce.{map|reduce}.memory.mb去解决问题。 但同时带来的问题是集群并行跑的container的数量少了，所以适当的调整内存参数对集群的利用率的提升尤为重要。

因为在yarn container这种模式下，JVM进程跑在container中，mapreduce.{map|reduce}.java.opts能够通过Xmx设置JVM最大的heap的使用，一般设置为0.75倍的memory.mb，因为需要为java code，非JVM内存使用等预留些空间

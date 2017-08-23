
>HDFS架构和GFS大致相同，Hadoop1.x版本的HDFS可以看做Google的GFS的简化版，因为HDFS绕去了GSF一写复杂的方案而采用简单方案。但Hadoop1.x版本的HDFS依旧存在单点失效和扩展性能不好的情况。
因此在Hadoop2.x版本通过高可用方案（HA）和NameNode联盟解决这两个问题，HA是解决主控服务器单点失效，NameNode联盟解决扩展性能。

## HDFS整体架构
HDFS整体架构由NameNode、DataNode、Secondary NameNode和客户端组成。
NameNode类似于GFS的“主控服务器”，DataNode类似于GFS的“Chunk服务器”。

### NameNode
NameNode负责整个文件系统的元数据，包括文件目录树结构、文件到数据块Block的映射关系、Block副本及其存储位置等各种管理数据。

这些数据**保存在内存**中，同时在磁盘上保存两个元数据管理文件：fsimage和editlog。
- fsimage：内存命名空间元数据在外存的镜像文件
- editlog：各种元数据操作的write-ahead-log文件，在体现到内存数据变化前先将操作写入editlog防止数据丢失。

这两个文件结合可以构造出完整的内存数据。

NameNode还负责监控DataNode的状态，二者通过短时间的心跳来传递管理信息和数据信息。

### Secondary NameNode
Secondary NameNode并非是NameNode的热备机，而是定期从NameNode那里拉取fsimage和editlog进行合并，形成新的fsimage传回给NameNode。目的是减轻NameNode的工作压力，NameNode本身并不进行这种合并操作，所以Secondary NameNode是一个提供检查点服务的服务器。

### DataNode
DataNode负责数据块的实际存储和读写工作，HDFS下称为Block，每个Block大小默认是64M，很多在线系统一般会改为128M。当客户端上传一个文件时，HDFS会切割成固定大小的Block。每个Block会多备份，默认是3个备份。

### 客户端
HDFS不支持客户端同时对一个文件进行并发写操作，同一时刻，只能有一个客户端在文件末尾进行追加写操作。

## HA
Hadoop2.x采用主从服务器模式来达到高可用，主控服务器由Active NameNode（ANN）和Standby NameNode（SNN）一主一从两个服务节点构成。ANN是客户端响应服务器，SNN作为冷备或者热备服务器。

当ANN发生故障时，由SNN接管工作，因此要求SNN元数据要和ANN保持一致。

为了达到元数据保持一致，HA方案通过以下两点来保证：
1. 使用第三方共享存储保存目录文件和命名空间元数据（editlog）。ANN将元数据的更改不断写入第三方共享存储，然后SNN通过第三方存储获取更新并体现在内存数据中，以此来达到数据同步。但是第三方存储可能存在单点失效情况，因此只是从NN的单点失效转移到别处。一般第三方有较强的容错和冗余机制，相比单服务器来说还是健壮一些。
2. 所有DataNode将心跳信息发给ANN和SNN，此时保证了数据一致性，但是还没有解决故障转移。HA将采用独立于NN的故障切换控制器（Failover Controller，FC），FC将监控NN服务器的硬件、操作系统及本身的健康状态信息，向Zookeeper写入心跳信息，由Zk做服务器领导者选举操作。

为了防止故障切换过程中出现脑裂，脑裂是指有2个或多个活跃主控服务器，HA需要进行一些必要的隔离措施：
- 第三方共享存储：保证在任意时刻，只有一个NN写入
- DataNode：保证只有一个NN发出与管理数据副本有关的删除命令
- 客户端：保证同一时刻只能有一个NN能够对客户端请求发出正确响应

上述HA有明显2个缺点：一是单点故障问题依旧可能发生，二是需要多处进行隔离以防止脑裂情况发生。

Cloudera在其Hadoop发行版中提供了QJM（Quorum Journal Manager）的HA方案。本质上是用Paxos协议在多个备份服务器进行主控服务器选举。QJM在`$2f+1$`个JournalNode上存储NN的editlog，每次写入操作，如果有`$f$`台服务器返回成功即可认为写入成功，通过Paxos来保证数据一致性。QJM允许同时最多`$F$`个JournalNode发生故障，而系统可以正常运行。

QJM相比Hadoop2.x提供的HA方案，有以下优势：
1. 彻底解决了单点问题，且可容忍最大故障JournalNode个数可以通过配置进行管理
2. 无需额外第三方存储设备，减少系统复杂和维护成本
3. 无需防止脑裂而单独在多处采用隔离措施，QJM本身内置了此功能

## 一、下载Hadoop
下载地址比较多，这里列出2个地址，以防止有些地址失效。
### 1. 方式一

```
wget http://www.motorlogy.com/apache/hadoop/common/current/hadoop-2.3.0.tar.gz
```

若不可访问的话，可在官网链接中找更多链接地址（http://www.apache.org/dyn/closer.cgi/hadoop/common/）可能早期的版本已经不存在了。
 ###   2. 方式二
可以在这个网址[（https://archive.cloudera.com/cdh5/cdh/5/）](https://archive.cloudera.com/cdh5/cdh/5/)下找到hadoop的安装包，也可以通过复制某个具体的tar.gz包使用wget方式获取，或者直接下载。当然这个网址不仅仅提供了Hadoop，而且提供了Zookeeper、Hbase、Hive、Hue、Impala、Pig、Solr、Spark等二进制包可下载使用。
    
下载完成之后，就可以将hadoop的包进行解压缩到指定目录（习惯放在/usr/local下）

```
tar xfz hadoop-2.3.0.tar.gz -C /usr/local/
```

然后建立一个软连接

```
ln -s hadoop-2.3.0 hadoop
```

    
## 二、Hadoop配置
Hadoop的配置主要会修改如下几个hadoop相关文件
-  /etc/profile    全局配置（用于JDK和hadoop配置），或者hadoop用户下的 ~/.bashrc文件，指针对某个用户生效
- /usr/local/hadoop/etc/hadoop/hadoop-env.sh
- /usr/local/hadoop/etc/hadoop/core-site.xml
- /usr/local/hadoop/etc/hadoop/hdfs-site.xml
- /usr/local/hadoop/etc/hadoop/mapred-site.xml (默认路径下是.template结尾的，将其改为.xml)
-  /usr/local/hadoop/etc/hadoop/yarn-site.xml（2.x使用YARN替代了JobTracker等资源管理）
- masters    配置masters文件
- slaves        配置slaves文件
 ###  1. 配置环境变量
 将Hadoop也设置进环境变量，这样就比较方便使用一些Hadoop的命令，同样在/etc/profile文件下追加如下内容

```
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
```

使用source /etc/profile使得修改立即生效

 ###   2.  配置hadoop-env.sh
这个文件主要是将其内的JAVA_HOME更改为已经安装的JDK目录

```
export JAVA_HOME=/usr/local/jdk
```

以下配置，可放在集群中，集群中所有节点配置相同。
###    3. core-site.xml
configuration节点下进行如下配置：

```
<configuration>
        <property>  
                <name>fs.default.name</name>  
                <value>hdfs://14.17.103.216:9000</value>  
        </property> 
        <!-- 指定hadoop运行时产生文件的存储目录 -->
        <property>
                <name>hadoop.tmp.dir</name>            
                <!-- 自定义目录 -->
                <!--    <value>/data1/zhaohui/hadoop/tmp</value> -->
        </property>
</configuration>
```

    1). fs.default.name是NameNode的URL，其值的格式是：hdfs://host:port
    2). hadoop.tmp.dir是hadoop的默认临时路径，若此配置进行了更改或者删除，需要重新执行格式化namenode命令。
 ###   4. hdfs-site.xml配置
    configuration节点下进行如下配置

```
<configuration>
        <property>  
                <!--开启web hdfs-->  
                <name>dfs.webhdfs.enabled</name>  
                <value>true</value>  
        </property>  
        <!-- 指定HDFS副本数量 -->
        <property>  
                <name>dfs.replication</name>  
                <value>2</value>  
        </property>
        <property>
                <name>dfs.namenode.checkpoint.dir</name>
                <value>/web/hadoop/hdfs/namesecondary</value>
                <description></description>
        </property>
        <property>  
                <name>dfs.namenode.name.dir</name>  
                <value>/data1/hadoop/data/dfs/name</value>  
                <description> namenode 存放name table(fsimage)本地目录</description>  
        </property>  
        <property>  
                <name>dfs.namenode.edits.dir</name>  
                <value>/data1/zhaohui/hadoop/data/dfs/edits</value>  
                <description>namenode存放 transaction file(edits)本地目录</description>  
        </property>  
        <property>  
                <name>dfs.datanode.data.dir</name>  
                <value>/data1/hadoop/data/dfs/data</value>  
                <description>datanode存放block本地目录</description>  
        </property>  
</configuration>
```

    1). dfs.webhdfs.enabled，用来开启web hdfs，此选项可以不设置
    2). dfs.replication，指定数据需要备份数量，若不设置，默认为3。若其值大于集群的SLAVE节点数，则会报错。
    3). dfs.namenode.name.dir，用来存储name空间的本地文件路径，多个值可用英文逗号分隔，hadoop会对name相关进行所有目录的冗余备份。
    4). dfs.namenode.edits.dir，用来存放name管理的事务日志本地文件路径，多个值可用英文逗号分隔，hadoop会对name相关进行所有目录的冗余备份。
    5). dfs.datanode.data.dir，DataNode存放块数据的目录，数据的真实存放目录，多个值用英文逗号分隔。数据会被存放在所有指定目录下。
###    5. mapred-site.xml配置

```
<configuration>
 <!-- 指定mr运行在yarn上 -->
  <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
  </property>
</configuration>
    1). mapreduce.framework.name，配置Mapreduce的运行框架为yarn
```
### 6. yarn-site.xml配置

```
<configuration>
  <property>
     <name>yarn.resourcemanager.hostname</name>
     <value>hadoop-master</value>
  </property>
  <property>
     <name>yarn.nodemanager.aux-services</name>
     <value>mapreduce_shuffle</value>
  </property>　　
</configuration>
```



    1). yarn.resourcemanager.hostname，配置resourceManager的地址，可以使用master的地址，或者制定另外一个机器作为yarn的地址
    2). yarn.nodemanager.aux-services，配置reducer获取数据的方式
    
    其他参数：
    - yarn.resourcemanager.address:yarn框架中NodeManager与RM通信的接口地址
    - yarn.resourcemanager.scheduler.address:同上，NodeManager需要知道RM主机的scheduler调度服务接口地址
    - yarn.resourcemanager.webapp.address:yarn框架中各个task的资源调度及运行状况通过该web界面访问
    - yarn.resourcemanager.resource-tracker.address:yran框架中NodeManager需要向RM报告任务运行状态供Resource跟踪，因此NodeManager节点主机需要知道RM主机的tracker接口地址
    
###    7. Masters文件配置
    此配置用来对master地址进行配置，可以使用域名方式或者IP地址方式，多个Master，每个占一行

```
vim masters
hadoop.master  #或者使用master的IP,比如：172.26.67.6
```

###    8. Slaves文件配置（Master主机特有）
此配置用来对slaves地址进行配置，同样可以使用域名主机方式或IP地址方式，多个slave，每个占一行

```
vim slaves
hadoop.slave1  #或者 172.26.67.5
hadoop.slave2  #或者 172.26.67.4
```

注：slaves文件配置在Slave机器上是可以没有的。
    当Master节点配置完成后，Slave节点和Master是一毛一样的，所以可以使用scp命令将hadoop文件夹复制到Slave机器上的相同目录。
    复制完成后，将Hadoop的HADOOP_HOME加入到环境变量中，见第一步。

## 三、启动hadoop
###    1. 格式化namenode
第一次必须要这么进行操作。格式化所有的数据都会没有，所以一般只第一次执行，会创建一些name配置需要的文件。

```
hadoop namenode -format
```

格式化成功后，会生成配置文件制定的目录，但是要注意创建文件需要hadoop有权限才行。
###    2. 启动hadoop节点
有2种方式启动
        
方式1，在hadoop的sbin文件夹下有一个start-all.sh文件，执行此文件，将会启动所有的hadoop节点

```
hadoop@vm-heron:/usr/local/hadoop/sbin$ ./start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
16/11/23 17:51:59 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [hadoop.master]
The authenticity of host 'hadoop.master (172.26.67.3)' can't be established.
ECDSA key fingerprint is 15:05:10:7a:a7:fb:c6:7a:1d:22:cc:f6:85:06:8d:3e.
Are you sure you want to continue connecting (yes/no)? yes
hadoop.master: Warning: Permanently added 'hadoop.master,172.26.67.3' (ECDSA) to the list of known hosts.
hadoop.master: starting namenode, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/hadoop-hadoop-namenode-vm-heron.out
localhost: starting datanode, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/hadoop-hadoop-datanode-vm-heron.out
hadoop.slave2: starting datanode, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/hadoop-hadoop-datanode-hadoop.out
hadoop.slave1: starting datanode, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/hadoop-hadoop-datanode-yy-vm.out
Starting secondary namenodes [0.0.0.0]
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is 15:05:10:7a:a7:fb:c6:7a:1d:22:cc:f6:85:06:8d:3e.
Are you sure you want to continue connecting (yes/no)? yes
0.0.0.0: Warning: Permanently added '0.0.0.0' (ECDSA) to the list of known hosts.
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/hadoop-hadoop-secondarynamenode-vm-heron.out
16/11/23 17:52:33 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/yarn-hadoop-resourcemanager-vm-heron.out
hadoop.slave2: starting nodemanager, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/yarn-hadoop-nodemanager-hadoop.out
localhost: starting nodemanager, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/yarn-hadoop-nodemanager-vm-heron.out
hadoop.slave1: starting nodemanager, logging to /usr/local/hadoop-2.3.0-cdh5.0.1/logs/yarn-hadoop-nodemanager-yy-vm.out
```

可以看到，先启动master的namenode、DataNode、Slave2的DataNode、Slave1的DataNode、然后启动secondary namenodes、resourcemanager、
    Slave2的nodemanager、Slave1的nodemanager
        通过jps命令进行验证

```
$ jps
4358 DataNode
4823 NodeManager
4233 NameNode
4569 SecondaryNameNode
5213 Jps
4703 ResourceManager
```

可以看到再Master上有5个节点启动。
        到Slave机器上jps下看是否有DataNode启动，若是没有启动，是防火墙的原因，在/etc/selinux/目录下，有个conf的配置文件，不同的操作系统可能名称不同，打开并追加以下内容，然后stop-all.sh杀掉所有进程后，在start-all.sh就可以了。

```
vim /etc/selinux/smanager.conf
selinux=disable
```

方式2，启动hadoop集群只需要启动HDFS集群和Map/Reduce集群
            在分配NameNode节点的机器上，同样执行在安装的sbin目录下如下命令：

```
/usr/local/hadoop/sbin/start-dfs.sh
```

在分配YARN节点的机器上，同样执行在安装的sbin目录下如下命令：

```
/usr/local/hadoop/sbin/start-yarn.sh
```


## 四、访问Hadoop
###    1. 访问NameNode
    
```
http://hadoop.master:50070
```
默认端口是50070。可以看到如下信息，Overview标签下是集群的整体状况，Datanodes标签下可以看到部署的Slave的DataNode状况。
        
![hadoop](C:/Users/Administrator/Documents/My%20Knowledge/temp/bfeeb67f-5c74-4047-b19f-1541f0c7d04e/4/index_files/891d4635-487f-42fb-b9c2-d1e3fe152ef5.png)


 ###    2. 任务应用（JobTracker）查看

```
http://hadoop.master:8088
```
默认端口是8088。可以看到如下内容
![image](C:/Users/Administrator/Documents/My%20Knowledge/temp/bfeeb67f-5c74-4047-b19f-1541f0c7d04e/4/index_files/00d459c5-2329-4a69-8395-5d5ac4fb3ccb.png)
 Cluster下可以看到关于集群操作的内容，Tools下可以看到配置信息以及日志信息。

## 五、测试
###   1. HDFS测试
上传文件到HDFS的根目录

```
hadoop fs -put xxx.tar.gz hdfs://hadoop.master:9000
```

可以先使用ls查看有哪些目录

```
hadoop fs -ls hdfs://hadoop.master:9000/
```

若想传到指定文件夹，但是文件夹必须存在，若不存在，可以使用下面命令创建一个
        接受路径指定的uri作为参数，创建这些目录。其行为类似于Unix的mkdir -p，它会创建路径中的各级父目录。

```
hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
```
 #在默认master配置机器上创建dir1目录和dir2目录

```
hadoop fs -mkdir hdfs://hadoop.master:9000/user/hadoop/dir hdfs://hadoop.host2:port2/user/hadoop/dir
```
   #明确指定在各主master上创建文件
### 2. MapReduce测试
        share/hadoop/mapreduce/下的example包，
        官方自带的Jar包使用用例测试说明
        
Valid program names are:
-   aggregatewordcount: An Aggregate based map/reduce program that counts the words in the input files.
-   aggregatewordhist: An Aggregate based map/reduce program that computes the histogram of the words in the input files.
-   bbp: A map/reduce program that uses Bailey-Borwein-Plouffe to compute exact digits of Pi.
-   dbcount: An example job that count the pageview counts from a database.
-   distbbp: A map/reduce program that uses a BBP-type formula to compute exact bits of Pi.
-   grep: A map/reduce program that counts the matches of a regex in the input.
-   join: A job that effects a join over sorted, equally partitioned datasets
-   multifilewc: A job that counts words from several files.
-   pentomino: A map/reduce tile laying program to find solutions to pentomino problems.
-   pi: A map/reduce program that estimates Pi using a quasi-Monte Carlo method.
-   randomtextwriter: A map/reduce program that writes 10GB of random textual data per node.
-   randomwriter: A map/reduce program that writes 10GB of random data per node.
-   secondarysort: An example defining a secondary sort to the reduce.
-   sort: A map/reduce program that sorts the data written by the random writer.
-   sudoku: A sudoku solver.
-   teragen: Generate data for the terasort
-   terasort: Run the terasort
-   teravalidate: Checking results of terasort
-   wordcount: A map/reduce program that counts the words in the input files.
-   wordmean: A map/reduce program that counts the average length of the words in the input files.
-   wordmedian: A map/reduce program that counts the median length of the words in the input files.
-   wordstandarddeviation: A map/reduce program that counts the standard deviation of the length of the words in the input files.
        使用简单的例子（测试pi）进行测试：

```
hadoop@vm-heron:/usr/local/hadoop$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.3.0-cdh5.0.1.jar pi 5 5
Number of Maps  = 5
Samples per Map = 5
16/11/24 16:55:12 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Starting Job
16/11/24 16:55:16 INFO client.RMProxy: Connecting to ResourceManager at hadoop.master/172.26.67.3:8032
16/11/24 16:55:18 INFO input.FileInputFormat: Total input paths to process : 5
16/11/24 16:55:18 INFO mapreduce.JobSubmitter: number of splits:5
16/11/24 16:55:19 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1479977560305_0001
16/11/24 16:55:20 INFO impl.YarnClientImpl: Submitted application application_1479977560305_0001
16/11/24 16:55:20 INFO mapreduce.Job: The url to track the job: http://hadoop.master:8088/proxy/application_1479977560305_0001/
16/11/24 16:55:20 INFO mapreduce.Job: Running job: job_1479977560305_0001
16/11/24 16:55:35 INFO mapreduce.Job: Job job_1479977560305_0001 running in uber mode : false
16/11/24 16:55:35 INFO mapreduce.Job:  map 0% reduce 0%
16/11/24 16:56:42 INFO mapreduce.Job:  map 20% reduce 0%
16/11/24 16:56:43 INFO mapreduce.Job:  map 100% reduce 0%
16/11/24 16:57:20 INFO mapreduce.Job:  map 100% reduce 100%
16/11/24 16:57:23 INFO mapreduce.Job: Job job_1479977560305_0001 completed successfully
16/11/24 16:57:25 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=116
		FILE: Number of bytes written=531129
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=1340
		HDFS: Number of bytes written=215
		HDFS: Number of read operations=23
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=3
	Job Counters 
		Launched map tasks=5
		Launched reduce tasks=1
		Data-local map tasks=5
		Total time spent by all maps in occupied slots (ms)=351903
		Total time spent by all reduces in occupied slots (ms)=25362
		Total time spent by all map tasks (ms)=351903
		Total time spent by all reduce tasks (ms)=25362
		Total vcore-seconds taken by all map tasks=351903
		Total vcore-seconds taken by all reduce tasks=25362
		Total megabyte-seconds taken by all map tasks=360348672
		Total megabyte-seconds taken by all reduce tasks=25970688
	Map-Reduce Framework
		Map input records=5
		Map output records=10
		Map output bytes=90
		Map output materialized bytes=140
		Input split bytes=750
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=140
		Reduce input records=10
		Reduce output records=0
		Spilled Records=20
		Shuffled Maps =5
		Failed Shuffles=0
		Merged Map outputs=5
		GC time elapsed (ms)=4786
		CPU time spent (ms)=5290
		Physical memory (bytes) snapshot=1017589760
		Virtual memory (bytes) snapshot=11267031040
		Total committed heap usage (bytes)=719736832
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=590
	File Output Format Counters 
		Bytes Written=97
Job Finished in 128.714 seconds
Estimated value of Pi is 3.68000000000000000000
```


Hadoop集群重启过程

## 接口总结

- Hadoop 集群的运行状态 http://192.168.199.201:8088/cluster

- Hadoop  集群的资源消耗查看http://192.168.199.201:50070

- Spark web控制界⾯面:http://192.168.199.201:8080/

- Chrongrapf Web接⼝口: http://222.200.180.199:8888

- Kibana，用ELK收集日志 http://222.200.180.178:5601/

  



## 启动Hadoop

```
# 开启hadoop
# 如果节点有变化，需要修改slaves文件
# cd /root/Hadoop/hadoop-2.7.7/etc/hadoop
# vim slaves
# 手动将slaves文件通过scp同步到每一个节点上
cd /root/Hadoop/hadoop-2.7.7/sbin/
./start-all.sh
```



现在Hadoop集群的网络结构如下：

- master 201
- slave 202-205



[Hadoop日志位置解析](https://blog.csdn.net/u011414200/article/details/50338073#11-hadoop系统服务输出的日志)

[Hadoop日志详解](https://blog.csdn.net/wangkai_123456/article/details/88937703)

[Hadoop系统角色分析](https://www.cnblogs.com/qingyunzong/p/8554869.html)



## 启动Spark：

```
# 开启spark
# 开启spark需要在hadoop开启后一段时间再弄，不然会报错（大概5min？）
# 如果节点有变化，需要修改slaves文件
# cd /root/Spark/spark-2.4.3-bin-hadoop2.7/conf
# vim slaves
# 手动将slaves文件通过scp同步到每一个节点上
cd /r
./start-all.sh
```



重启其中一个spark节点（注意Spark监听端口是7077）

在Spark/sbin目录下执行命令：

```
./start-slave.sh 192.168.199.201:7077
```

来源：https://blog.51cto.com/beyond3518/1787498



## 启动InfluxDB

InfluxDB监控着⼀一系列列 `time series` ，每个 `time series `包含⼀一系列的 points ，其中每个点包含以下：

- time :时间戳

- measurement :测量量的内容，如cpu_load
- 至少一个 field ，是 measurement 的值，如value=0.64或temperature=21.2
- 零到多个 tags ，包含任何关于值的metadata，如host=server01, region=EMEA, dc=Frankfurt

这个属性可以继续添加



InfluxDB在222.200.180.199

Kibana http://222.200.180.178:5601/, 这个是用来看日志的界面，用的是ELK收集日志

成功启动InfluxDB后，就可以在Chrongrapf Web接⼝http://222.200.180.199:8888查看可视化后的KPI时序数据。



## 启动HiBench

HiBench是一个大数据基准套件，可以帮助评测不同大数据平台的性能、吞吐量和系统资源利用率。它包含一组Hadoop、Spark和Streaming测试模式，包含Sort、WordCount、TeraSort、Sleep、SQL、PageRank、Nutch index、Bayes、Kmeans、NWeight和增强型的DFSIO等。现在集群中应该是已经安装了HadoopBench和SparkBench的环境。

### 目录解释 

![image-20200214145208742](/Users/leexy/Library/Application Support/typora-user-images/image-20200214145208742.png)



- autogen：主要用于生成测试数据的源码目录

- bin：测试脚本放置目录

- common：公共依赖源码目录

- conf：配置文件目录（Hibench/Hadoop/Spark等配置文件存放目录）
- docker：
- flinkbench:Flink框架源码目录
- gearpumpbench：gearpumpbench框架源码目录
- hadoopbench：hadoop框架源码目录
- sparkbench：spark框架的源码目录
- stormbench：storm框架的源码目录



### 运行任务

- 运行批量任务，编辑` /root/HiBench/conf/benchmarks.lst` ，列列举全部要运⾏的任务，并编辑 `/root/HiBench/conf/frameworks.lst` 设置要运⾏的框架(Hadoop、 Spark...)然后执⾏ `/root/HiBench/bin/run_all.sh`

![img](https://ask.qcloudimg.com/http-save/yehe-1522219/2tooqvaco5.jpeg?imageView2/2/w/1620)



- 持续运行，执行`/root/HiBench/bin/run_forever.sh`

## Kibana

阿里提供 Kibana，您可以对自己的 Elasticsearch （全文搜索）进行可视化，还可以在 Elastic Stack 中进行导航，这样您便可以进行各种操作了，从跟踪查询负载，到理解请求如何流经您的整个应用，都能轻松完成。ElasticSearch+filebeat

日志收集的配置文件：`/etc/filebeat/filebeat.yaml`，可以修改配置来收集不同node的日志。



现在已安装filebeat的节点：

- 192.168.199.201，收集的是namenode
- 192.168.199.202





## 故障注入

在本次实验，使用chaosblade故障注入工具。官方文档https://chaosblade-io.gitbook.io/chaosblade-help-zh-cn/

目前在201和202机器上装了chaosblade，路径`/root/FaultInjection/chaosblade-0.4.0`



故障考虑几点：

- cpuHog
- MemHog
- Network delay
- reproduce 其他paper或者hadoop里面的bug

!!!超级重要，注入network delay的时候记得指定端口和故障时间，不然会导致远程服务器无法连接





## 实验过程

### 补充之前的实验

1. 不同的logparser之间的对比

2. 



实验过程：

1. 用不同的parser进行parse `StackReduceV1\parser`
2. 进行train test的数据集分离 `LogRobust/HDFS_data_generator.py`
3. 用LogRobustTrain.py的train 
4. 用`shuf`命令来打乱顺序，再切片成小数据集进行test
5. 成功后，进行LogRobustTest.py的test





## Anomaly Detection Part 实验





### fail-stop的实验数据产生

- crash的数据

  

### performance bug的实验数据产生

- cpu 高占用 CPUhog

- 内存 高占用 Memhog

- network delay, NetworkJam

- reproduce bug index:

  - hang performance bug
    - Continuously read until timeout on a socket. (#HDFS-3318)https://issues.apache.org/jira/browse/HDFS-3318
    - Hedged read might hang infinitely if read data from all DN failedhttps://issues.apache.org/jira/browse/HDFS-11303?jql=project%20%3D%20HDFS%20AND%20text%20~%20hang
    - webhdfs create a file and send buffersize=0 hangs the requesthttps://issues.apache.org/jira/browse/HDFS-2469?jql=project%20%3D%20HDFS%20AND%20text%20~%20hang
  - slowdown performance bug(这里的思想主要是找到需要return的一些方法注入delay或者sleep一阵子)
    - 注入delay的

- 两种faults：

  - external faults（现在我们注入的是external faults，外界cpuUsage，memUsage，NetworkJam）
  - internal faults（接下来在Java程序中注入internal fauls），chaosblade可以做到
    - 注入delay：https://github.com/hopshadoop/hops/blob/master/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/server/datanode/fsdataset/impl/FsDatasetAsyncDiskService.java
    - Datanodehttps://github.com/bruthe/hadoop-2.6.0r/blob/master/src/hdfs/org/apache/hadoop/hdfs/server/datanode/DataNode.java

  ```sh
  # delete file service
  ./blade create dubbo delay --time 3000 --service org.apache.hadoop.hdfs.server.datanode.fsdataset.impl.FsDatasetAsyncDiskService --methodname=deleteAsync --provider	
  
  # datanode block receiver service
  ./blade create dubbo delay --time 3000 --service org.apache.hadoop.hdfs.server.datanode.DataNode.BlockReceiver --methodname=receiveBlock --provider	
  
  # datanode block packet responder service
  ./blade create dubbo delay --time 3000 --classname org.apache.hadoop.hdfs.server.datanode.DataNode.BlockReceiver.PacketResponder --methodname=run --provider	
  ```

org.apache.hadoop.hdfs.server.datanode.DataNode.BlockReceiver.PacketResponder


Noting：在运行过程中，Hadoop和Spark会产生大量的结果数据，若要持续运行，一定要记得定时删除这些结果数据。

Hadoop:`~/.tmp`临时数据

Spark:`$Spark/works/app*`









## 遇到的坑

### 运行HiBench的workload一直卡在running job

![image-20200217215637503](/Users/leexy/Library/Application Support/typora-user-images/image-20200217215637503.png)

![image-20200217215651949](/Users/leexy/Library/Application Support/typora-user-images/image-20200217215651949.png)

原因：各节点一直处于Unhealthy的状态，修改检测Unhealthy的标准

![image-20200218133538551](/Users/leexy/Library/Application Support/typora-user-images/image-20200218133538551.png)

解决方法：修改yarn-site.xml配置

https://blog.csdn.net/zengmingen/article/details/52609873

！https://www.jianshu.com/p/5dbb9011d2ee

Yarn简单介绍及内存配置https://blog.csdn.net/zengmingen/article/details/52609893

```html
<property> 
	<name>yarn.nodemanager.disk-health-checker.min-healthy-disks</name>              	 <value>0.0</value>
</property> 
<property> 
	<name>yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</name>
 <value>100.0</value>
</property>
```

### 删除冗余block

在各节点的`/tmp`中会存放处理的data block，当任务执行完后，我们可以删掉他们来释放磁盘空间。删除后再次启动Hadoop的任务就会遇到下面的错误：

![image-20200218211631356](/Users/leexy/Library/Application Support/typora-user-images/image-20200218211631356.png)

此时Hadoop进入安全模式。什么是Hadoop的安全模式呢？ 
在分布式文件系统启动的时候，开始的时候会有安全模式，当分布式文件系统处于安全模式的情况下，文件系统中的内容不允许修改也不允许删除，直到安全模式结束。安全模式主要是为了系统启动的时候检查各个DataNode上数据块的有效性，同时根据策略必要的复制或者删除部分数据块。运行期通过命令也可以进入安全模式。在实践过程中，系统启动的时候去修改和删除文件也会有安全模式不允许修改的出错提示，只需要等待一会儿即可。 

用一下命令可以离开safe mode

```
bin/hadoop dfsadmin -safemode leave
```



![image-20200218223254628](/Users/leexy/Library/Application Support/typora-user-images/image-20200218223254628.png)

## 实验结果记录

### fail-stop的对比实验



#### Attn + Bi-LSTM

- Drain:
  - TF-IDF:

  ![image-20200217171820001](/Users/leexy/Library/Application Support/typora-user-images/image-20200217171820001.png)

  - BERT:

  ![image-20200217171844913](/Users/leexy/Library/Application Support/typora-user-images/image-20200217171844913.png)

- AEL:
  - TF-IDF:

  ![image-20200218155741041](/Users/leexy/Library/Application Support/typora-user-images/image-20200218155741041.png)

  - BERT:

  ![image-20200218171846960](/Users/leexy/Library/Application Support/typora-user-images/image-20200218171846960.png)

- LogInsight:
  - TF-IDF: 5
  
    ![image-20200221141719970](/Users/leexy/Library/Application Support/typora-user-images/image-20200221141719970.png)
  
  - BERT:
  
    ![image-20200221141915156](/Users/leexy/Library/Application Support/typora-user-images/image-20200221141915156.png)

####attn + 单向LSTM

- Drain：

  - TF-IDF：![image-20200219201133753](/Users/leexy/Library/Application Support/typora-user-images/image-20200219201133753.png)
  - bert：![image-20200219201147419](/Users/leexy/Library/Application Support/typora-user-images/image-20200219201147419.png)

- AEL：

  - TF-IDF：

    ![image-20200219201218136](/Users/leexy/Library/Application Support/typora-user-images/image-20200219201218136.png)

  - bert：

    ![image-20200219201238275](/Users/leexy/Library/Application Support/typora-user-images/image-20200219201238275.png)

- LogInsight8
  
  - TF-IDF：
  
    ![image-20200221183448306](/Users/leexy/Library/Application Support/typora-user-images/image-20200221183448306.png)
  
  - BERT：
  
    <img src="/Users/leexy/Library/Application Support/typora-user-images/image-20200221183436907.png" alt="image-20200221183436907" style="zoom:200%;" />

#### attn + 不同的Bi-LSTM

- LogInsight：
  - TF-IDF：![image-20200224170017384](/Users/leexy/Library/Application Support/typora-user-images/image-20200224170017384.png)
  - BERT：![image-20200224165945435](/Users/leexy/Library/Application Support/typora-user-images/image-20200224165945435.png)

noattn+lstm

![image-20200303171439017](/Users/leexy/Library/Application Support/typora-user-images/image-20200303171439017.png)

错误注入纪录：

- CpuHog
- ![image-20200219172816867](/Users/leexy/Library/Application Support/typora-user-images/image-20200219172816867.png)

- MemHog

![image-20200219172752165](/Users/leexy/Library/Application Support/typora-user-images/image-20200219172752165.png)



TODO：

- [x] 安装（不知道是否已经安装）并找到InfluxDB的所在地，启动InfluxDB
- [x] 找到log的所在地
- [x] 安装chaosblade
- [x] 将HiBench正常跑起来，收集正常的数据
- [x] 修改AEL的产生过程
- [ ] 进行有无Attention
- [ ] 找到Hadoop report的issue进行reproduce
- [x] 有无Bi的对比实验(差LogInsight)
- [ ] 改进attn+Bi-LSTM模型
- [ ] 处理数据





![image-20200225151018117](/Users/leexy/Library/Application Support/typora-user-images/image-20200225151018117.png)

![image-20200225151210296](/Users/leexy/Library/Application Support/typora-user-images/image-20200225151210296.png)





### performance bug

### attn 后拼接

 Precision: 0.674%, Recall: 0.761%, F1-measure: 0.545%



## data fusion

 Precision: 66.675%, Recall: 77.797%, F1-measure: 54.425%



修改注入错误后

Precision: 0.379%, Recall: 0.638%, F1-measure: 0.115%

Precision: 0.660%, Recall: 0.700%, F1-measure: 0.401%



数据集shuffle后：

 Precision: 0.907, Recall: 0.992, F1-measure: 0.947



perf_2: 8-12，50%-60%

Precision: 0.957%, Recall: 0.996%, F1-measure: 0.976%



Perf_3: 按照相关比例进行错误注入

 Precision: 0.963%, Recall: 0.995%, F1-measure: 0.979%



Perf_4: 随机注入5-6条，8-12秒

 Precision: 0.935, Recall: 0.995, F1-measure: 0.962



Perf_5: 随机注入5-6条，1s

Precision: 0.903, Recall: 0.986, F1-measure: 0.941



perf_without_injection: baseline 看看没有注错的结果有多少

 Precision: 0.656%, Recall: 0.664%, F1-measure: 0.355%



Perf_6: 在0-2秒的日志上增加1s延时，2-3条日志

 Precision: 0.830%, Recall: 0.984%, F1-measure: 0.893%



perf_5, lstm

Precision: 89.724%, Recall: 98.521%, F1-measure: 93.684%



Perf_5, bi-lstm

Precision: 84.144%, Recall: 98.473%, F1-measure: 89.984%



Perf_5, bert, attn-bi-lstm

Precision: 91.710%, Recall: 98.654%, F1-measure: 94.885%

Precision: 89.065%, Recall: 98.510%, F1-measure: 93.328%



perf_5, mean:

Precision: 67.294%, Recall: 93.994%, F1-measure: 76.075%



Perf_5:,只对一维time操作

 Precision: 67.010%, Recall: 68.011%, F1-measure: 41.812%



Perf_5: 不进行倒数操作

 Precision: 65.188%, Recall: 66.421%, F1-measure: 55.700%



敏感性测试：

对应的是哪些performance issue呢？

1. 变化的个数【1，2，3，4，5】
2. 变化的幅度【0.1, 0.3, 0.5, a1，2，5，10，20】

| dataset                | encoding | bi   | attn | result                                                   |
| ---------------------- | -------- | ---- | ---- | -------------------------------------------------------- |
| Perf_5                 | W2v      | T    | T    | Precision: 90.3%, Recall: 98.6%, F1-measure: 94.1%       |
| Perf_5                 | Bert     | T    | T    | Precision: 91.710%, Recall: 98.654%, F1-measure: 94.885% |
| Perf_5                 | w2v      | T    | F    | Precision: 84.144%, Recall: 98.473%, F1-measure: 89.984% |
| Perf_5                 | w2v      | F    | F    | Precision: 89.724%, Recall: 98.521%, F1-measure: 93.684% |
| Perf_2                 | W2v      | T    | T    | Precision: 95.7%, Recall: 99.6%, F1-measure: 97.6%       |
| perf_2                 | bert     | T    | T    | Precision: 92.856%, Recall: 99.405%, F1-measure: 95.900% |
| Perf_2                 | W2v      | T    | F    | Precision: 95.226%, Recall: 99.518%, F1-measure: 97.293% |
| Perf_2                 | W2v      | F    | F    | Precision: 95.012%, Recall: 99.618%, F1-measure: 97.225% |
| Perf_3                 | W2v      | T    | T    | Precision: 96.3%, Recall: 99.5%, F1-measure: 97.9%       |
| Perf_3                 | bert     | T    | T    | Precision: 93.459%, Recall: 99.141%, F1-measure: 96.157% |
| Perf_3                 | W2v      | T    | F    | Precision: 95.575%, Recall: 99.560%, F1-measure: 97.479% |
| Perf_3                 | W2v      | F    | F    | Precision: 94.658%, Recall: 99.284%, F1-measure: 96.796% |
| Perf_4                 | W2v      | T    | T    | Precision: 93.5%, Recall: 99.5%, F1-measure: 96.2%       |
| perf_4                 | bert     | T    | T    | Precision: 91.955%, Recall: 98.638%, F1-measure: 95.095% |
| Perf_4                 | W2v      | T    | F    | Precision: 96.606%, Recall: 99.496%, F1-measure: 97.994% |
| Perf_4                 | W2v      | F    | F    | Precision: 92.303%, Recall: 99.589%, F1-measure: 95.666% |
| Perf_6                 | W2v      | T    | T    | Precision: 83.0%, Recall: 98.4%, F1-measure: 89.3%       |
| Perf_6                 | bert     | T    | T    | Precision: 86.064%, Recall: 98.116%, F1-measure: 91.113% |
| Perf_6                 | W2v      | T    | F    | Precision: 82.705%, Recall: 98.159%, F1-measure: 88.510  |
| Perf_6                 | W2v      | F    | F    | Precision: 82.660%, Recall: 98.175%, F1-measure: 88.484% |
| Perf_without_injection | W2v      | T    | T    | Precision: 65.6%, Recall: 66.4%, F1-measure: 35.5%       |
| perf_without_injection | bert     | T    | T    | Precision: 62.700%, Recall: 66.299%, F1-measure: 64.369% |
| Perf_without_injection | W2v      | T    | F    | Precision: 64.450%, Recall: 66.535%, F1-measure: 65.456% |
| Perf_without_injection | W2v      | F    | F    | Precision: 64.552%, Recall: 66.666%, F1-measure: 57.012% |
|                        |          |      |      |                                                          |





W2v

| cnt  | second    | Result                                                   |
| ---- | --------- | -------------------------------------------------------- |
| 1    | 1         | Precision: 67.940%, Recall: 91.842%, F1-measure: 71.659% |
| 1    | 1 bi-lstm | Precision: 71.909%, Recall: 92.110%, F1-measure: 75.582% |
| 1    | 1 lstm    | Precision: 73.452%, Recall: 91.807%, F1-measure: 77.715% |
| 2    | 1         | Precision: 78.937%, Recall: 97.602%, F1-measure: 85.015% |
| 2    | 1 bi-lstm | Precision: 85.829%, Recall: 97.418%, F1-measure: 90.699% |
| 2    | 1 lstm    | Precision: 86.704%, Recall: 97.710%, F1-measure: 91.355% |
| 3    | 1         | Precision: 88.750%, Recall: 99.111%, F1-measure: 93.406% |
| 3    | 1 bi-lstm | Precision: 89.520%, Recall: 99.111%, F1-measure: 93.894% |
| 3    | 1 lstm    | Precision: 87.615%, Recall: 99.183%, F1-measure: 92.651% |
| 5    | 1         | Precision: 93.817%, Recall: 99.276%, F1-measure: 96.396% |
| 5    | 1 bi-lstm | Precision: 92.623%, Recall: 99.418%, F1-measure: 95.758% |
| 5    | 1 lstm    | Precision: 94.779%, Recall: 99.611%, F1-measure: 97.092% |
| 1    | 2         | Precision: 83.566%, Recall: 97.117%, F1-measure: 88.879% |
| 1    | 2 bi-lstm | Precision: 79.388%, Recall: 96.870%, F1-measure: 85.804% |
| 1    | 2 lstm    | Precision: 82.288%, Recall: 96.930%, F1-measure: 87.852% |
| 2    | 2         | Precision: 86.004%, Recall: 99.046%, F1-measure: 91.555% |
| 2    | 2 bi-lstm | Precision: 87.957%, Recall: 99.192%, F1-measure: 92.977% |
| 2    | 2 lstm    | Precision: 93.302%, Recall: 99.380%, F1-measure: 96.183% |
| 3    | 2         | Precision: 89.215%, Recall: 99.547%, F1-measure: 93.779% |
| 3    | 2 bi-lstm | Precision: 92.372%, Recall: 99.606%, F1-measure: 95.731% |
| 3    | 2 lstm    | Precision: 90.163%, Recall: 94.891%, F1-measure: 92.415% |
| 5    | 2         | Precision: 95.620%, Recall: 99.761%, F1-measure: 97.600% |
| 5    | 2 bi-lstm | Precision: 97.922%, Recall: 99.736%, F1-measure: 98.812% |
| 5    | 2 lstm    | Precision: 97.040%, Recall: 99.699%, F1-measure: 98.319% |



bert

| cnt  | second    | Result                                                   |
| ---- | --------- | -------------------------------------------------------- |
| 1    | 1         | Precision: 73.889%, Recall: 91.386%, F1-measure: 77.895% |
| 1    | 1 bi-lstm | Precision: 72.527%, Recall: 91.392%, F1-measure: 75.693% |
| 1    | 1 lstm    | Precision: 73.843%, Recall: 91.338%, F1-measure: 77.284% |
| 2    | 1         | Precision: 80.287%, Recall: 97.442%, F1-measure: 85.766% |
| 2    | 1 bi-lstm | Precision: 82.209%, Recall: 97.634%, F1-measure: 87.889% |
| 2    | 1 lstm    | Precision: 83.516%, Recall: 97.651%, F1-measure: 88.563% |
| 3    | 1         | Precision: 93.652%, Recall: 98.876%, F1-measure: 96.130% |
| 3    | 1 bi-lstm | Precision: 90.864%, Recall: 98.609%, F1-measure: 94.326% |
| 3    | 1 lstm    | Precision: 92.103%, Recall: 99.063%, F1-measure: 95.321% |
| 5    | 1         | Precision: 94.321%, Recall: 99.611%, F1-measure: 96.848% |
| 5    | 1 bi-lstm | Precision: 96.309%, Recall: 99.671%, F1-measure: 97.942% |
| 5    | 1 lstm    | Precision: 96.750%, Recall: 99.743%, F1-measure: 98.209% |
| 1    | 2         | Precision: 82.410%, Recall: 97.019%, F1-measure: 87.759% |
| 1    | 2 bi-lstm | Precision: 85.302%, Recall: 96.560%, F1-measure: 89.949% |
| 1    | 2 lstm    | Precision: 78.117%, Recall: 96.415%, F1-measure: 84.515% |
| 2    | 2         | Precision: 90.961%, Recall: 98.858%, F1-measure: 94.601% |
| 2    | 2  lstm   | Precision: 88.798%, Recall: 99.196%, F1-measure: 93.516% |
| 2    | 2 bi-lstm | Precision: 89.958%, Recall: 99.194%, F1-measure: 94.177% |
| 3    | 2         | Precision: 93.427%, Recall: 99.625%, F1-measure: 96.305% |
| 3    | 2 bi-lstm | Precision: 93.444%, Recall: 99.648%, F1-measure: 96.356% |
| 3    | 2 lstm    | Precision: 95.003%, Recall: 99.679%, F1-measure: 97.232% |
| 5    | 2         | Precision: 96.748%, Recall: 99.775%, F1-measure: 98.190% |
| 5    | 2 bi-lstm | Precision: 94.911%, Recall: 99.768%, F1-measure: 97.224% |
| 5    | 2 lstm    | Precision: 98.213%, Recall: 99.746%, F1-measure: 98.964% |




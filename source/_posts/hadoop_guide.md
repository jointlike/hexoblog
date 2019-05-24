
---
title: hadoop 使用
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: 大数据
tags: ['大数据','hadoop']
---
## 模块与概念解释

### 存储组件
+ 存储方式
- 列式存储，整个文件被切割为若干列数据，每一列数据一起存储。文件类型有：Parquet , RCFile, ORCFile。面向列的格式使得读取数据时，可以跳过不需要的列，适合于只处于行的一小部分字段的情况。但是这种格式的读写需要更多的内存空间，因为需要缓存行在内存中（为了获取多行中的某一列）。同时不适合流式写入，因为一旦写入失败，当前文件无法恢复，而面向行的数据在写入失败时可以重新同步到最后一个同步点，所以Flume采用的是面向行的存储格式。

- 行式存储，同一行的数据存储在一起，即连续存储。文件类型有：SequenceFile, MapFile, Avro Datafile。采用这种方式，如果只需要访问行的一小部分数据，亦需要将整行读入内存，推迟序列化一定程度上可以缓解这个问题，但是从磁盘读取整行数据的开销却无法避免。面向行的存储适合于整行数据需要同时处理的情况。其中，Avro是一个数据序列化系统。序列化就是将对象转换成二进制流,相应的反序列化就是将二进制流再转换成对应的对象，设计用于支持大批量数据交换的应用。

+ 存储结构
HBase是一个针对结构化数据的可伸缩、高可靠、高性能、分布式和面向列的动态模式数据库。和传统关系数据库不同，HBase采用了BigTable的数据模型：增强的稀疏排序映射表（Key/Value），其中，键由行关键字、列关键字和时间戳构成。HBase提供了对大规模数据的随机、实时读写访问，同时，HBase中保存的数据可以使用MapReduce来处理，它将数据存储和并行计算完美地结合在一起。

## hadoop数据存储
 需要着重考虑以下几个问题：数据存储格式，多租户（支持多个用户、多个组以及多种应用类型），模式设计（输入输出的目录结构等及HBase hive数据结构），元数据管理。存储的选择关系到数据的安全，计算的效率，及存储的效益。

1. 数据存储选择
 - 文件格式选择（考虑：是否可压缩，是否可分片，序列化方式，列式存储或行式存储格式，SequenceFile、Avro 等容器格式是文本等大部分文件类型的优选格式）
 - 压缩格式选择（压缩比例，是否支持分片，计算时需要的转换消耗，分记录级压缩、块级压缩）
 - 数据存储系统选择（HBase, HDFS）
 
 一般来说，在与容器文件格式（Avro、SequenceFile 等）一起使用时，任何压缩格式都可以是分片式的，因为容器文件格式能够单独压缩记录构成的数据块，也可以进行记录级的压缩。如果在压缩整个文件时没有使用容器文件格式，那么就需要使用本身支持可分片的压缩格式，比如在数据块之间插入同步标记的 bzip2

### 查询计算组件
+ hadoop HSQL、hbaseSQL、impala、pig计算框架
分布式离线计算框架主要适用于大批量的集群任务，由于是批量执行，故时效性偏低。

- hive
A data warehouse infrastructure that provides data summarization and ad hoc querying.
Hive定义了一种类似SQL的查询语言(HQL),将SQL转化为MapReduce任务在Hadoop上执行。通常用于离线分析。

- Impala 相对于hive的实时查询系统，它提供SQL语义抛弃了MapReduce这个不太适合做SQL查询的范式，能查询存储在Hadoop的HDFS和HBase中的PB级大数据。已有的Hive系统虽然也提供了SQL语义，但由于Hive底层执行使用的是MapReduce引擎，仍然是一个批处理过程，难以满足查询的交互性。相比之下，Impala的最大特点也是最大卖点就是它的快速，启动速度和省去磁盘IO。

- pig
A high-level data-flow language and execution framework for parallel computation.
定义了一种数据流语言—Pig Latin，将脚本转换为MapReduce任务在Hadoop上执行。通常用于进行离线分析。

- Mahout
Mahout起源于2008年，最初是Apache Lucent的子项目，它在极短的时间内取得了长足的发展，现在是Apache的顶级项目。Mahout的主要目标是创建一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。Mahout现在已经包含了聚类、分类、推荐引擎（协同过滤）和频繁集挖掘等广泛使用的数据挖掘方法。除了算法，Mahout还包含数据的输入/输出工具、与其他存储系统（如数据库、MongoDB 或Cassandra）集成等数据挖掘支持架构。

+ solr 搜索计算框架
- solr Apache Solr是一个功能强大的搜索服务器，它支持REST风格API。Solr是基于Lucene的，Lucene 支持强大的匹配能力，如短语，通配符，连接，分组和更多不同的数据类型。

+ spark 准实时框架 
Spark 是专为大规模数据处理而设计的快速通用的计算引擎，其是基于内存的迭代式计算。

- Spark Core：包含Spark的基本功能；尤其是定义RDD的API、操作以及这两者上的动作。其他Spark的库都是构建在RDD和Spark Core之上的
- Spark SQL：提供通过Apache Hive的SQL变体Hive查询语言（HiveQL）与Spark进行交互的API。每个数据库表被当做一个RDD，Spark SQL查询被转换为Spark操作。
- Spark Streaming：对实时数据流进行处理和控制。Spark Streaming允许程序能够像普通RDD一样处理实时数据
- MLlib：一个常用机器学习算法库，算法被实现为对RDD的Spark操作。这个库包含可扩展的学习算法，比如分类、回归等需要对大量数据集进行迭代的操作。
- GraphX：控制图、并行图操作和计算的一组算法和工具的集合。GraphX扩展了RDD API，包含控制图、创建子图、访问路径上所有顶点的操作Spark架构的组成图如下

+ Storm实时框架
Storm是一个分布式的、可靠的、容错的流式计算框架。Storm 一开始就是为实时处理设计，因此在实时分析/性能监测等需要高时效性的领域广泛采用。Storm在理论上支持所有语言，只需要少量代码即可完成适配。

### CDH管理监控组件
- Cloudera Manager. 监控管理组件。
- Navigator是企业版的。CDH 平台的端到端数据管理工具。Cloudera Navigator 使管理员、数据经理和分析师能够了解 Hadoop 中的大量数据。

### 通用集群管理组件
- Zookeeper
分布式协作服务，Zookeeper是google Chubby克隆版,解决分布式环境下的数据管理问题：统一命名，状态同步，集群管理，配置同步等。

- Sqoop
数据同步工具，Sqoop是SQL-to-Hadoop的缩写，主要用于传统数据库和Hadoop之间传输数据， 数据的导入和导出本质上是Mapreduce程序，充分利用了MR的并行化和容错性。




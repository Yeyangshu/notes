# Flink API

## 1 Flink抽象API

Flink中提供了 4 种不同层次的 API，如图所示：

- 低级 API：提供了对时间和状态的细粒度控制，简洁性和易用性较差，主要应用在对一些复杂事件的处理逻辑上。
- 核心 API：主要提供了针对流数据和离线数据的处理，对低级 API 进行了一些封装，提供了filter、sum、max、min 等高级函数，简单且易用，所以在工作中应用比较广泛。
  - DataSet 处理有界的数据集
  - DataStream 处理有界或者无界的数据流。
- Table API：一般与 DataSet 或者 DataStream 紧密关联，首先通过一个 DataSet 或 DataStream 创建出一个 Table；然后用类似于 filter、join 或者 select 关系型转化操作来转化为一个新的 Table 对象；最后将一个 Table 对象转回一个 DataSet 或 DataStream。与 SQL 不同的是，Table API 的查询不是一个指定的 SQL 字符串，而是调用指定的 API 方法。
- SQL：Flink 的 SQL 集成是基于 Apache Calcite 的，Apache Calcite 实现了标准的 SQL，使用起来比其他 API 更加灵活，因为可以直接使用 SQL 语句。Table API 和 SQL 可以很容易地结合在一块使用，它们都返回 Table 对象。

![image-20210309224148710](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210309224148710.png)

## 2 Flink DataStream 的常用 API

DataStream API 主要分为3块：DataSource、Transformation、Sink。

- DataSource 是程序的数据源输入，可以通过 `StreamExecutionEnvironment.addSource(sourceFunction)` 为程序添加一个数据源。

  ![image-20210309230649789](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210309230649789.png)

- Transformation 是具体的操作，它对一个或多个输入数据源进行计算处理，比如 Map、FlatMap 和 Filter 等操作。

- Sink 是程序的输出，它可以把 Transformation 处理之后的数据输出到指定的存储介质中。

### 2.1 DataSource

Flink 针对 DataStream 提供了大量的已经实现的 DataSource（数据源）接口。

#### 2.1.1 基于文件

读取文本、HDFS文件创建一个数据源，文件遵循TextInputFormat逐行读取规则并返回。

```java
StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
// 文件地址
environment.readFile("hdfs://node01:9000/flink/data")
```

#### 2.1.2 基于Socket

从 Socket 中读取数据，元素可以通过一个分隔符分开。

```java
environment.socketTextStream
```

#### 2.1.3 基于集合

通过 Java 的 Collection 集合创建一个数据流，集合中的所有元素必须是相同类型的。一般用于测试场景，没有太大的实际意义。

```java
environment.fromCollection
```

#### 2.1.4 自定义输入

addSource可以实现读取第三方数据源的数据。Flink也提供了一批内置的Connector（连接器）。

连接器会提供对应的Source支持，如表所示。

![image-20210309231537486](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210309231537486.png)

自定义数据源，有两种方式实现。

- 通过实现SourceFunction接口来自定义无并行度（也就是并行度只能为1）的数据源。
- 通过实现ParallelSourceFunction 接口或者继承RichParallelSourceFunction 来自定义有并行度的数据源。

##### 2.1.4.1 自定义数据源练习

需求：实现并行度只能为1的自定义DataSource以及SourceFunction接口。

### 2.2 Transformation

Flink针对DataStream提供了大量的已经实现的算子。

- Map：输入一个元素，然后返回一个元素，中间可以进行清洗转换等操作。
- FlatMap：输入一个元素，可以返回零个、一个或者多个元素。
- Filter：过滤函数，对传入的数据进行判断，符合条件的数据会被留下。
- KeyBy：根据指定的Key进行分组，Key相同的数据会进入同一个分区。KeyBy的两种典型用法如下：
  - DataStream.keyBy("someKey") ：指定对象中的someKey段作为分组Key。
  - DataStream.keyBy(0) ：指定Tuple中的第一个元素作为分组Key。
- Reduce：对数据进行聚合操作，结合当前元素和上一次Reduce返回的值进行聚合操作，然后返回一个新的值。
- Aggregations：sum()、min()、max()等。
- Union：合并多个流，新的流会包含所有流中的数据，但是Union有一个限制，就是所有合并的流类型必须是一致的。
- Connect：和Union类似，但是只能连接两个流，两个流的数据类型可以不同，会对两个流中的数据应用不同的处理方法。
- coMap和coFlatMap：在ConnectedStream中需要使用这种函数，类似于Map和flatMap。
- Split：根据规则把一个数据流切分为多个流。
- Select：和Split配合使用，选择切分后的流。

Flink针对DataStream提供了一些数据分区规则

- Random partitioning：随机分区。

- Rebalancing：对数据集进行再平衡、重分区和消除数据倾斜。

- Rescaling：重新调节。

- Custom partitioning：自定义分区。

  自定义分区实现Partitioner接口的方法如下。

  ```java
  DataStream.partitionCustom(new MyPartition(), 0);
  ```

### 2.3 SinkFlink

针对DataStream提供了大量的已经实现的数据目的地（Sink），具体如下所示。

- writeAsText()：将元素以字符串形式逐行写入，这些字符串通过调用每个元素的toString()方法来获取。
- print() / printToErr()：打印每个元素的toString()方法的值到标准输出或者标准错误输出流中。
- 自定义输出：addSink可以实现把数据输出到第三方存储介质中。

可以自定义Sink，有两种实现方式。

- 实现SinkFunction接口。
- 继承RichSinkFunction类。

##  3 Flink DataSet的常用API分析

DataSet API 主要可以分为 3 块来分析：DataSource、Transformation和Sink。

- DataSource是程序的数据源输入。
- Transformation是具体的操作，它对一个或多个输入数据源进行计算处理，比如Map、FlatMap、Filter等操作。
- Sink是程序的输出，它可以把Transformation处理之后的数据输出到指定的存储介质中。

### 3.1 DataSource

#### 3.1.1 基于集合

fromCollection(Collection)，主要是为了方便测试使用。

#### 3.1.2 基于文件

readTextFile(path)，基于HDFS中的数据进行计算分析。

### 3.2 Transformation

Flink针对DataSet提供了大量的已经实现的算子。

- Map：输入一个元素，然后返回一个元素，中间可以进行清洗转换等操作。
- FlatMap：输入一个元素，可以返回零个、一个或者多个元素。
- MapPartition：类似Map，一次处理一个分区的数据（如果在进行Map处理的时候需要获取第三方资源连接，建议使用MapPartition）。
- Filter：过滤函数，对传入的数据进行判断，符合条件的数据会被留下。
- Reduce：对数据进行聚合操作，结合当前元素和上一次Reduce返回的值进行聚合操作，然后返回一个新的值。
- Aggregations：sum、max、min等。
- Distinct：返回一个数据集中去重之后的元素。
- Join：内连接。
- OuterJoin：外链接。
- Cross：获取两个数据集的笛卡尔积。
- Union：返回两个数据集的总和，数据类型需要一致。
- First-n：获取集合中的前N个元素。
- Sort Partition：在本地对数据集的所有分区进行排序，通过sortPartition()的链接调用来完成对多个字段的排序。

Flink针对DataSet提供了一些数据分区规则，具体如下：

- Rebalance：对数据集进行再平衡、重分区以及消除数据倾斜操作。
- Hash-Partition：根据指定Key的散列值对数据集进行分区。
- Range-Partition：根据指定的Key对数据集进行范围分区。
- Custom Partitioning：自定义分区规则，自定义分区需要实现Partitioner接口。

### 3.3 Sink

Flink针对DataSet提供了大量的已经实现的Sink。

- writeAsText()：将元素以字符串形式逐行写入，这些字符串通过调用每个元素的toString()方法来获取。
- writeAsCsv()：将元组以逗号分隔写入文件中，行及字段之间的分隔是可配置的，每个字段的值来自对象的toString()方法。
- print()：打印每个元素的toString()方法的值到标准输出或者标准错误输出流中。

## 4 Flink Table API和SQL的分析及使用

Flink针对标准的流处理和批处理提供了两种关系型API：Table API和SQL。

- TableAPI允许用户以一种很直观的方式进行select、filter和join操作；
- Flink SQL支持基于 Apache Calcite实现的标准SQL。针对批处理和流处理可以提供相同的处理语义和结果。

Flink的Table API和SQL是捆绑在Flink-Table依赖中的，如果项目中想要使用Table API和SQL，就必须要添加下面依赖。

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table_2.11</artifactId>
    <version>1.6.1</version>
</dependency>
```

Table API和SQL通过join API集成在一起，这个join API的核心概念是Table，Table可以作为查询的输入和输出。

### 4.1 Table API和SQL的基本使用

想使用Table API和SQL，首先要创建一个TableEnvironment。TableEnvironment对象是Table API和SQL集成的核心，通过TableEnvironment可以实现以下功能。

- 通过内部目录创建表。
- 通过外部目录创建表。
- 执行SQL查询。
- 注册一个用户自定义的Function。
- 把DataStream或者DataSet转换成Table。
- 持有ExecutionEnvironment或者StreamExecutionEnvironment的引用。

一个查询中只能绑定一个指定的TableEnvironment，TableEnvironment可以通过Table Environment.getTableEnvironment()或者TableConfig来生成。TableConfig可以用来配置TableEnvironment或者自定义查询优化。

创建一个TableEnvironment对象

```java
// 流数据查询
StreamExecutionEnvironment sEnv = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment sTableEnv = TableEnvironment.getTableEnvironment(sEnv);

// 批数据查询
ExecutionEnvironment bEnv = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment bTableEnv = TableEnvironment.getTableEnvironment(bEnv);
```

通过获取到的TableEnvironment对象可以创建Table对象，有两种类型的Table对象：输入Table(Input Table)和输出Table(Output Table)。

- 输入Table可以给Table API和SQL提供查询数据
- 输出Table可以把Table API和SQL的查询结果发送到外部存储介质中。

输入Table可以通过多种数据源注册。

- 已存在的Table对象：通常是Table API和SQL的查询结果。
- TableSource：通过它可以访问外部数据，比如文件、数据库和消息队列。
- DataStream或DataSet。输出Table需要使用TableSink注册。

如何通过TableSource注册一个Table。
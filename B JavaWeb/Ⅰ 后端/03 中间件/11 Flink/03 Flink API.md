# Flink API

## 1 Flink抽象API

Flink中提供了 4 种不同层次的 API，如图所示：

- 低级 API：提供了对时间和状态的细粒度控制，简洁性和易用性较差，主要应用在对一些复杂事件的处理逻辑上。
- 核心 API：主要提供了针对流数据和离线数据的处理，对低级 API 进行了一些封装，提供了filter、sum、max、min 等高级函数，简单且易用，所以在工作中应用比较广泛。
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

读取文本文件，文件遵循TextInputFormat逐行读取规则并返回。

```java
StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();

environment.readFile
```

#### 2.1.2 基于Socket

从Socket中读取数据，元素可以通过一个分隔符分开。

```java
environment.socketTextStream
```

#### 2.1.3 基于集合

通过Java的Collection集合创建一个数据流，集合中的所有元素必须是相同类型的。

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


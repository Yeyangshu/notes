# Elasticsearch基本概念

官网中文教程：https://www.elastic.co/guide/cn/index.html

Elasticsearch: 权威指南：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

Elasticsearch：**分布式、高性能、高可用、可伸缩、易维护**

Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，

## 1 基础入门与常见术语

*Elasticsearch* 是一个实时的分布式搜索分析引擎，它能让你以前所未有的速度和规模，去探索你的数据。 它被用作全文检索、结构化搜索、分析以及这三个功能的组合。

Elasticsearch的一些常见术语。

- **Index**：Elasticsearch的Index相当于数据库的Table
- **Type**：这个在新的Elasticsearch版本已经废除（在以前的Elasticsearch版本，一个Index下支持多个Type--有点类似于消息队列一个topic下多个group的概念）
- **Document**：Document相当于数据库的一行记录
- **Field**：相当于数据库的Column的概念
- **Mapping**：相当于数据库的Schema的概念
- **DSL**：相当于数据库的SQL（给我们读取Elasticsearch数据的API）

![preview](https://pic3.zhimg.com/v2-814655822e4782d6277b7bc0b0c420d6_r.jpg)

## 2 安装并运行Elasticsearch

1. 去官网下载对应的 Elasticsearch 版本： https://www.elastic.co/cn/downloads/elasticsearch

2. 解压文件至指定目录

3. 启动 Elasticsearch

   ```powershell
   cd elasticsearch-<version>
   # 前台启动
   ./bin/elasticsearch
   # 后台启动
   ./bin/elasticsearch -d
   ```

4. 测试是否启动

   ```powershell
   curl 'http://localhost:9200/?pretty'
   
   # 如果返回如下信息，就意味着你现在已经启动并运行一个 Elasticsearch 节点。
   {
     "name" : "Tom Foster",
     "cluster_name" : "elasticsearch",
     "version" : {
       "number" : "2.1.0",
       "build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
       "build_timestamp" : "2015-11-18T22:40:03Z",
       "build_snapshot" : false,
       "lucene_version" : "5.3.1"
     },
     "tagline" : "You Know, for Search"
   }
   
   ```

完整的请求：`curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'`

![image-20210303235101654](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210303235101654.png)

样例：

```shell
curl -H "Content-Type: application/json" -XGET 'localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'

# 响应
{
  "count" : 0,
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "skipped" : 0,
    "failed" : 0
  }
}
```

## 3 面向文档

Elasticsearch 是 *面向文档* 的，意味着它存储整个对象或 *文档*。Elasticsearch 不仅存储文档，而且 *索引* 每个文档的内容，使之可以被检索。在 Elasticsearch 中，我们对文档进行索引、检索、排序和过滤—而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

## 4 Elasticsearch CRUD 练习

### 4.1 需求：创建一个雇员目录

我们受雇于 *Megacorp* 公司，作为 HR 部门新的 *“热爱无人机”* （*"We love our drones!"*）激励项目的一部分，我们的任务是为此创建一个员工目录。该目录应当能培养员工认同感及支持实时、高效、动态协作，因此有一些业务需求：

- 支持包含多值标签、数值、以及全文本的数据
- 检索任一员工的完整信息
- 允许结构化搜索，比如查询 30 岁以上的员工
- 允许简单的全文搜索以及较复杂的短语搜索
- 支持在匹配文档内容中高亮显示搜索片段
- 支持基于数据创建和管理分析仪表盘

### 4.2 索引（INSERT）员工文档

文档地址：https://www.elastic.co/guide/cn/elasticsearch/guide/current/_indexing_employee_documents.html#_indexing_employee_documents

第一个业务需求是存储员工数据。 这将会以 *员工文档* 的形式存储：一个文档代表一个员工。存储数据到 Elasticsearch 的行为叫做 *索引* ，但在索引一个文档之前，需要确定将文档存储在哪里。

对于员工目录，我们将做如下操作：

- 每个员工索引（动词，INSERT）一个文档，文档包含该员工的所有信息。
- 每个文档都将是 `employee` *类型* 。
- 该类型位于 *索引*（名词，数据库） `megacorp` 内。
- 该索引保存在我们的 Elasticsearch 集群中。

命令

```powershell
curl -H "Content-Type: application/json" -XPUT 'localhost:9200/megacorp/employee/1' -d '
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}'

# 响应
{"_index":"megacorp","_type":"employee","_id":"1","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

### 4.3 检索指定雇员的数据

将 HTTP 命令由 `PUT` 改为 `GET` 可以用来检索文档，同样的，可以使用 `DELETE` 命令来删除文档，以及使用 `HEAD` 指令来检查文档是否存在。如果想更新已存在的文档，只需再次 `PUT` 。

检索 `id=1` 雇员的数据

命令：

```powershell
curl -H "Content-Type: application/json" -XGET 'localhost:9200/megacorp/employee/1'

# 响应
{"_index":"megacorp","_type":"employee","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}}
```

### 4.4 轻量搜索

 `GET` 是相当简单的，可以直接得到指定的文档。

#### 4.4.1 搜索所有雇员：

使用索引库 `megacorp` 以及类型 `employee`，但与指定一个文档 ID 不同，这次使用 `_search` 。返回结果包括了所有三个文档，放在数组 `hits` 中。一个搜索默认返回十条结果。

注意：返回结果不仅告知匹配了哪些文档，还包含了整个文档本身：显示搜索结果给最终用户所需的全部信息。

```powershell
curl -H "Content-Type: application/json" -XGET 'localhost:9200/megacorp/employee/_search'

# 响应
{
    "took": 114, 
    "timed_out": false, 
    "_shards": {
        "total": 1, 
        "successful": 1, 
        "skipped": 0, 
        "failed": 0
    }, 
    "hits": {
        "total": {
            "value": 1, 
            "relation": "eq"
        }, 
        "max_score": 1, 
        "hits": [
            {
                "_index": "megacorp", 
                "_type": "employee", 
                "_id": "1", 
                "_score": 1, 
                "_source": {
                    "first_name": "John", 
                    "last_name": "Smith", 
                    "age": 25, 
                    "about": "I love to go rock climbing", 
                    "interests": [
                        "sports", 
                        "music"
                    ]
                }
            }
        ]
    }
}
```

#### 4.4.2 高亮搜索

尝试下搜索姓氏为 ``Smith`` 的雇员。为此，我们将使用一个 *高亮* 搜索，很容易通过命令行完成。这个方法一般涉及到一个 *查询字符串* （*query-string*） 搜索，因为我们通过一个URL参数来传递查询信息给搜索接口。

在请求路径中使用 `_search` 端点，并将查询本身赋值给参数 `q=` 。返回结果给出了所有的 Smith：

命令

```powershell
curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith&pretty"

# 响应
{
  "took" : 45,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "first_name" : "John",
          "last_name" : "Smith",
          "age" : 25,
          "about" : "I love to go rock climbing",
          "interests" : [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}

```

#### 4.4.3 查询表达式搜索




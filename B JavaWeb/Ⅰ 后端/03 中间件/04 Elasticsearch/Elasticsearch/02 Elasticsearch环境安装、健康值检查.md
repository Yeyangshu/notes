# 1 安装

1、**安装环境**（Windows和Linux）

**(1)** **安装ES**

①  安装六字箴言：

1)   JDK->依赖

2)   下载->elastic.co

3)   启动->./elasticsearch -d

4)   验证->http://localhost:9200/

②  开发模式和生产模式

1)   开发模式：默认配置（未配置发现设置），用于学习阶段

2)   生产模式：会触发ES的引导检查，学习阶段不建议修改集群相关的配置。

**(2)** **安装Kibana（**从版本6.0.0开始，Kibana仅支持64位操作系统。**）**

①  **下载**：http://elastic.co

②  **启动**：依然是开箱即用 

​           **Linux**：./kibana

   **Windows**：.\kibana.bat

③  **验证**：localhost:5601

(3)  **安装Head插件（选装）**：

①  **介绍**：提供可视化的操作页面对ElasticSearch搜索引擎进行各种设置和数据检索功能，可以很直观的查看集群的健康状况，索引分配情况，还可以管理索引和集群以及提供方便快捷的搜索功能等等。

②  **下载**：https://github.com/mobz/elasticsearch-head同时课件中也提供了安装包。

③  **安装**：依赖于node和grunt管理工具

④  **启动**：npm run start

⑤  **验证**：http://localhost:9100/

# 2 集群健康值

(1)  健康值检查

①  _cat/health

②  _cluster/health

(2)  健康值状态

①  Green：所有Primary和Replica均为active，集群健康

②  Yellow：至少一个Replica不可用，但是所有Primary均为active，数据仍然是可以保证完整性的。

③  Red：至少有一个Primary为不可用状态，数据不完整，集群不可用。

 

 
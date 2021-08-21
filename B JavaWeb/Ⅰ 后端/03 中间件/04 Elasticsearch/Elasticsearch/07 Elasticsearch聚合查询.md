# 聚合查询

## 1 bucket 和 metirc

## 2 语法

```json
GET /product/_search
{
  "aggs": {
    "NAME": {
      "AGG_TYPE": {}
    }
  }
}
```

## 3 goup by

1. 以tag维度每个产品的数量，即每个标签

    ```json
    GET /product/_search
    {
      "query": {
        "match_all": {}
      }
    }
    
    # 以tag维度每个产品的数量，即每个标签
    # 只用tag是无法查询的，因为是text类型，需要使用.keyword
    GET /product/_search
    {
      "aggs": {
        "tag_agg_group": {
          "terms": {
            "field": "tags.keyword"
          }
        }
      },
      "size": 0
    }
    
    # 原product索引结构
    GET /product/_mapping
    {
      "product" : {
        "mappings" : {
          "properties" : {
            "desc" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "name" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "price" : {
              "type" : "long"
            },
            "tags" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            }
          }
        }
      }
    }
    
    # 或者设置text类型的fielddata类型为true
    PUT /product/_mapping
    {
      "properties": {
        "tags": {
          "type": "text",
          "fielddata": true
        }
      }
    }
    
    # 修改之后的索引结构
    GET /product/_mapping
    {
      "product" : {
        "mappings" : {
          "properties" : {
            "desc" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "name" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "price" : {
              "type" : "long"
            },
            "tags" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              },
              "fielddata" : true
            }
          }
        }
      }
    }
    
    # 修改之后就可以查询出来，但是性能很低
    GET /product/_search
    {
      "aggs": {
        "tag_agg_group": {
          "terms": {
            "field": "tags"
          }
        }
      },
      "size": 0
    }
    
    ```

2. 基础上增加筛选条件：统计价格大于1999的数据

    ```json
    GET /product/_search
     {
       "query": {
         "bool": {
           "filter": {
             "range": {
               "price": {
                 "gt": 1999
               }
             }
           }
         }
       },
       "aggs": {
         "tag_agg_group": {
           "terms": {
             "field": "tags.keyword"
           }
         }
       },
       "size": 0
     }
    ```

## 4 avg

1. 价格大于1999的每个tag产品的平均价格

    ```json
    GET /product/_search
    {
      "aggs": {
        "avg": {
          "terms": {
            "field": "tags.keyword",
            "order": {
              "avg_price": "desc"
            }
          },
          "aggs": {
            "avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      },
      "size": 0
    }
    ```

    结果：

    ```json
    {
      "took" : 2,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 5,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [ ]
      },
      "aggregations" : {
        "avg" : {
          "doc_count_error_upper_bound" : 0,
          "sum_other_doc_count" : 1,
          "buckets" : [
            {
              "key" : "gongjiaoka",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 4999.0
              }
            },
            {
              "key" : "buka",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 3999.0
              }
            },
            {
              "key" : "fashao",
              "doc_count" : 3,
              "avg_price" : {
                "value" : 3999.0
              }
            },
            {
              "key" : "xingjiabi",
              "doc_count" : 3,
              "avg_price" : {
                "value" : 3999.0
              }
            },
            {
              "key" : "menjinka",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 2999.0
              }
            },
            {
              "key" : "bufangshui",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 999.0
              }
            },
            {
              "key" : "low",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 999.0
              }
            },
            {
              "key" : "yinzhicha",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 999.0
              }
            },
            {
              "key" : "lowbee",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 399.0
              }
            },
            {
              "key" : "xuhangduan",
              "doc_count" : 1,
              "avg_price" : {
                "value" : 399.0
              }
            }
          ]
        }
      }
    }
    
    ```

## 5. 分组聚合

按照千元机：1000以下 ，中端机：2000-3000，高端机：3000以上分组聚合，分别计算数量

```json
GET /product/_search
{
  "aggs": {
    "tag_agg_group": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 100,
            "to": 1000
          },
          {
            "from": 1000,
            "to": 2000
          },
          {
            "from": 3000
          }
        ]
      },
      "aggs": {
        "price_agg": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  },
  "size": 0
}
```

结果

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "tag_agg_group" : {
      "buckets" : [
        {
          "key" : "100.0-1000.0",
          "from" : 100.0,
          "to" : 1000.0,
          "doc_count" : 2,
          "price_agg" : {
            "value" : 699.0
          }
        },
        {
          "key" : "1000.0-2000.0",
          "from" : 1000.0,
          "to" : 2000.0,
          "doc_count" : 0,
          "price_agg" : {
            "value" : null
          }
        },
        {
          "key" : "3000.0-*",
          "from" : 3000.0,
          "doc_count" : 2,
          "price_agg" : {
            "value" : 4499.0
          }
        }
      ]
    }
  }
}

```


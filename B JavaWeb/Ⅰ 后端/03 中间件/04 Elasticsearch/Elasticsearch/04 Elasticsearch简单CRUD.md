# CRUD

1. 创建索引：PUT /product?pretty

2. 查询索引：GET _cat/indices?v

3. 删除索引：DELETE /product?pretty

4. 插入数据：

   ```json
   PUT /index/_doc/id
   {
     Json数据
   }
   
   PUT /product/_doc/1
   {
       "name" : "xiaomi phone",
       "desc" :  "shouji zhong de zhandouji",
       "price" :  3999,
       "tags": [ "xingjiabi", "fashao", "buka" ]
   }
   
   ```

5. 更新数据

- 全量替换

  ```json
  PUT /index/_doc/id
  {
    全量Json数据
  }
  
  PUT /product/_doc/1
  {
      "name" : "xiaomi phone",
      "desc" :  "shouji zhong de zhandouji",
      "price" :  5999,
      "tags": [ "xingjiabi", "fashao", "buka" ]
  }
  ```

- 指定字段更新

  ```json
  PUT /index/_doc/id/_update
  {
    "doc": {
        "指定字段"： "新值"
    }
  }
  
  PUT /product/_doc/1/_update
  {
      "doc": {
          "price" :  5999
      }
  }
  ```

6. 删除数据 DELETE /index/type/id
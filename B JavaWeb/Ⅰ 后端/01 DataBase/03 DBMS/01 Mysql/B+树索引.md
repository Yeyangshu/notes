# B+树索引

## 1 B+树索引

B+ 索引在数据库中有一个特点是高扇出性，在数据库中，B+ 树的高度一般都在 2~4 层，这也说明查找某一键值的行记录最多需要 2 到 4 次 IO。当前一般的机械键盘每秒至少可以做 100 次 IO，2~4 次的 IO 意味着查询时间只需 0.02~0.04 秒。

数据库中的 B+ 树索引可以分为聚集索引（clustered index）和辅助索引（secondary index，辅助索引有时也称为非聚集索引（non-clustered index）），两者内部都是 B+ 树，即高度平衡的，叶子结点存放着所有的数据。聚集索引与辅助索引不同的是，叶子结点存放的是否是一整行的信息。

### 1.1 聚集索引

InnoDB 存储引擎表是索引组织表，即表中数据按照主键顺序存放。

**由于实际的数据页只能按照一颗 B+ 树进行排序，因此每张表只能拥有一个聚集索引。**

聚集索引的一个好处是，它对于主键的排序查找和范围查找速度非常快。叶子结点的数据就是用户所要查询的数据。

#### 1.1.1 主键排序查询

如用户需要查询一张注册用户的表，查询最后注册的 10 位用户，由于 B+ 树索引的双向链表，用户可以快速找到最后一个数据页，并取出 10 条记录。

```powershell
mysql> explain select * from t order by a limit 2\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.01 sec)
```

虽然使用 ORDER BY 对记录进行排序，但是在实际过程中并没有进行所谓的 filesort 操作。

#### 1.1.2 范围查询

如果要查找主键某一范围内的数据，通过叶子结点的上层中间节点就可以得到页的范围，之后直接读取数据页即可。

```powershell
mysql> explain select * from t where a > 1 and a < 3\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

### 1.2 辅助索引

辅助索引（Secondary Index，也称为非聚集索引），叶子结点并不包含行记录的全部数据。叶子节点除了包含键值以外，每个叶子节点的索引行中还包含了一个书签（bookmark）。辅助索引的书签就是相应行数据的聚集索引键。

200页截图

## 2 B+ 树的使用

### 2.1 联合索引

#### 2.1.1 联合索引的键值都是排序

联合索引的键值都是排序的，通过叶子节点可以逻辑上顺序地读出所有数据。

可以使用

```sql
SELECT * FROM TABLE WHERE a=xxx and b = xxx;
SELECT * FROM TABLE WHERE a=xxx;
```

不能使用

```sql
SELECT * FROM TABLE WHERE b=xxx;
```

#### 2.1.2 第二个键值进行排序

联合索引已经是排好序的，使用联合索引避免多一次排序操作。

对于联合索引（a, b, c）可以使用

```sql
SELECT ··· FROM TABLE WHERE a=xxx ORDER BY b;
SELECT ··· FROM TABLE WHERE a=xxx AND b = xxx ORDER BY b;
```

不能使用

```sql
SELECT ··· FROM TABLE WHERE a=xxx ORDER BY c;
```

### 2.2 覆盖索引

覆盖索引（covering index），即从辅助索引中就可以得到查询的记录，而不需要查询聚集索引中的记录。

使用覆盖索引的一个好处是辅助索引不包含整行的所有信息，故其大小远小于聚集索引，因此可以减少大量的 IO 操作。



乐观锁

```java
private static final int RETRY_TIMES = 3;
    private static final int INIT_VERSION = 1;

    /**
     * 基于version的乐观锁机制控制并发，重试三次
     * @param keyName
     */
    public <T> void addData(String keyName, T data) {
        ResultCode resultCode = null;

        // 重试次数
        for (int i = 0; i < RETRY_TIMES; i++) {
            resultCode = tryAddData(keyName, data);
            if (ResultCode.SUCCESS.equals(resultCode)) {
                return;
            }
        }

        log.error("put data fail, resultCode = ", resultCode);
    }

    /**
     * 添加数据
     * 在每次put之前，必须先做get，put的时候使用get结果携带的version作为参数传递。
     * 如果服务端原来没有此key，则不管传递什么version，都可以put成功，并且将此key对应的version更新为 1。
     * 如果get得到的结果为空，即服务端没有此key，则使用>= 2或者< 0的version做put，切不可使用0或者1作为version更新
     *
     * @param keyName 查询的key
     * @param data 想添加的数据
     * @return 添加结果
     */
    private <T> ResultCode tryAddData(String keyName, T data) {
        // 获得带版本的数据
        Result<DataEntry> existResult = dataService.getObjectWithVersionInfo(keyName);

        // 查询数据失败
        if (null != existResult && !existResult.isSuccess()) {
            return existResult.getResultCode();
        }

        // 合并数据的集合
        Queue<T> mergeResult = new LinkedList();

        // 查询成功，没有值
        if (null == existResult || null == existResult.getValue()) {
            mergeResult.add(data);
            // 初始化数据与版本号
            return dataService.putWithVersion(keyName, mergeResult, INIT_VERSION);
        }

        // 获取查询到的数据
        DataEntry entry = existResult.getValue();
        mergeResult.add(data);
        // 添加合并后的数据，以及查询到的版本号进行更新
        return dataService.putWithVersion(keyName, mergeResult, entry.getVersion());
    }
```


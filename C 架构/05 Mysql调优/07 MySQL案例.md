DB问题

- 一个是数据可靠性
- 一个是性能

# 慢sql优化案例

## 1 线上案例

### 1.1 背景

履行系统多级网络项目刚刚上线，为了支持多级作业，将运单和作业单做了分离，将原先的ct_order，拆分出nc_tms_order作业单。

刚上线很多业务场景还没有加索引，上线几天随着业务不断深入，终于爆发。

因为一个慢sql，导致DB服务器load上升，DB连接池耗尽，应用接口超时，甚至期间业务无法正常使用。

```sql
SELECT COUNT(1)
FROM `nc_tms_order`
WHERE `nc_tms_order`.`wh_id`= 108884
AND `nc_tms_order`.`order_type`= 1
AND `nc_tms_order`.`distribution_order_id`='12059380340'
AND `nc_tms_order`.`is_deleted`= 0
```

从IDB上看到，平均响应时间5s，扫描303531行，流量一旦上来，基本就打满整个线程池，db性能急剧下降。

开发同学想到的是加索引，下午17.10分通过加上索引后，DB的各项性能数据恢复。

如下图，新增唯一索引

```sql
idx_whid_distributionorderid(wh_id, distribution_order_id) 
```

通过这个索引解决线上故障，接下来分析下优化过程。



idb分析：

wh_id：唯一值数目 2

wh_id, distribution_order_id：唯一值数目2k



wh_id，总条目数是30万+，wh_id唯一值条目为2，按照公式count(distinct col)/count(*)，区分度基本等于0。



sql解析的额外信息，当出现using index时，表示sql使用覆盖索引，性能较好，而当出现using filesort、using temporary、using where时，查询需要优化。

### 1.2 案例分析

本次主要针对ct_order,tms_order等表进行优化，约影响索引20个。这里主要针对几个特例分析下。

#### 1.2.1 出库单原先1650ms 缩减到10ms

```sql
SELECT COUNT(1) FROM nc_tms_order `nc_tms_order` WHERE `nc_tms_order`.`STATUS` IN (?) AND `nc_tms_order`.`transfer_order_code` = ?AND `nc_tms_order`.`wh_id` = ?AND `nc_tms_order`.`gmt_arrived` >= ? AND `nc_tms_order`.`is_deleted` = ?
```

- 原索引结构：idx(wh_id,site_id,transfer_order_code)

- 优化后：idx(transfer_order_code)

很明显，原索引是一个无效索引，where条件中无site_id，按照索引最左前缀原则，那么原索引走到wh_id就停止了。

新索引删除掉了wh_id,site_id字段后，成功优化。一直就讲过，wh_id县仓id在我们这种场景下作为索引字段是很业余的做法，先不说600个县仓通过取模#wh_id#%1024，落到相同分区表的数据很少外(据统计一般一个wh_id一个分区表，少数情况有两个县仓在同一分区表)，依据建索引的另一个原则，选择性最大的字段放在最前面，也就是说site_id应该放在wh_id前,transfer_order_code放在site_id前，最后的索引理论应该是Idx(transfer_order_code,site_id,wh_id). 好了，这样看下来好像没啥毛病。

那么为什么我最终的索引是idx(transfer_order_code)呢，根据选择性最大原则，wh_id,的区分度是可以忽略的，这个是个理由，其实还有一个更重要的原因是，根据我们这个sql的场景，通过确定transfer_order_codes的值，实际上就已经确定了wh_id,site_id；transfer_order_codes只可能属于一个wh_id、site_id，通过transfer_order_codes确定后面的wh_id,site_id索引条目只能是唯一的,再建索引非常冗余，有画蛇添足之意。打个比方，你在地图上搜索乐佳国际(前提默认在杭州市范围内)，你不需要冗余的说明“中国浙江省杭州市某某区某某路乐佳国际大厦”，你只会告诉搜索引擎“乐佳国际”四个字，它就会搜索你想要的大厦。我想说的是乐佳国际本身就是一个区分度很高的关键字了不需要你在多此一举告诉计算机这是在杭州余杭区的大厦（实际同城内大厦名称也不可能有重复的）。这让我又想起了国外人写信地址是从小到大，从经验上讲是符合索引字段排列顺序的，还是比较合理的。

那么基于以上原因，我把所有不必要的wh_id、site_id都去掉了。当然我说的是不必要，那么情况下是必要的呢，比如说，我想要统计下某个站点下site_id的所有订单，那么这个时候，site_id作为索引字段是很必要。这个就好比，交易中，买家要查询自己所有已支付的订单，那么user_id即作为分区字段，同时也作为分区表的索引字段。

#### 1.2.2 2000ms领件缩减到20ms以内

```sql
SELECT COUNT(1) FROM `nc_tms_order_0609` `nc_tms_order`WHERE `nc_tms_order`.`STATUS` IN (3, 30)AND `nc_tms_order`.`SOP_TYPE` IN (1, 3, 5, 6)AND `nc_tms_order`.`wh_id` = 259681AND `nc_tms_order`.`dest_site_id` IN (266806)AND `nc_tms_order`.`gmt_arrived` >= '2017-08-14 00:00:00'AND `nc_tms_order`.`order_type` = 1AND `nc_tms_order`.`distribution_order_id` IS NULL AND `nc_tms_order`.`is_deleted` = 0
```

- 原索引：idx(wh_id,distribution_order_id)

- 优化后的索引：idx(distribution_order_id,status,dest_site_id)
  领件条件，distribution为空，唯一条目大概有6k，选择性不足，所以得加上status、dest_site_id，这个时候就能很快定位了。

#### 1.2.3 8000ms 大包 缩减到20ms以内

```sql
select *
FROM ct_order `ct_order`WHERE `ct_order`.`STATUS` IN (?)AND `ct_order`.`IS_DELETED` = ?AND `ct_order`.`GMT_ARRIVED` >= ?AND `ct_order`.`GMT_ARRIVED` <= ?AND `ct_order`.`SOP_TYPE` IN (?)AND `ct_order`.`STATION_ID` = ?AND `ct_order`.`ORDER_TYPE` = ?AND `ct_order`.`DISTRIBUTION_ORDER_ID` IS NULLORDER BY `ct_order`.`ID` DESCLIMIT ?,?
```

-  原索引：idx(wh_id,distribution_order_id)

-  优化后索引：idx(distribution_order_id,sop_type)

此索引与2索引其实是一个样的问题，为什么优化后的索引结构不同了，原因是应为根据这个sql场景，这里主要还是针对集包的领件，通过sop_type in (2) ，实际选择性很大，很快就能确定范围。这就是具体场景具体分析的案例。

## 2 建索引的几大原则

对于索引我总结三大原则：

1. 最左前缀原则；
2. 不冗余原则；
3. 最大选择性原则。

基本掌握这三条，对于索引的优化理论上是没有问题了。

### 2.1 最左前缀原则

#### 2.1.1 最左前缀是一个很重要的原则

MySQL会从左至右匹配，直到遇到范围查找(>、<、like、between)就停止，如：

```sql
select * from table1 where a=1 and b=2 and c<3 and d=9 ;
```

建立的联合索引为：(a, b, c, d) ，实际使用的索引为(a, b, c)。因为遇到了c<3就停止了，d列就没有用上。

前面讲过联合索引是有序元组，则MySQL实际建的索引为：(a)、(a,b)、(a,b,c)、(a,b,c,d)。

举个例子：where b=2 and c=3 and d=9 ；按照最左匹配原则，这个条件就没法走索引了，首先必须有a。

#### 2.1.2 =,in可以乱序

查询优化器会帮你优化成索引可以识别的形式。也就是说，where b = 2 and a = 1 and c<3，使用的索引任然为(a, b, c)组

### 2.2 不冗余原则

#### 2.2.1 尽量扩展索引、不要新建索引

mysql目前主要索引有：FULLTEXT，HASH，BTREE

好的索引可以提高我们的查询效率，不好的索引不但不会起作用，反而给DB带来负担，基于BTREE结构，插入、修改都会重新调整索引结构，存储成本增加，写效率降低，同时DB系统也要消耗资源去维护。

基于刚才的最左匹配原则，尽量在原有基础上扩展索引，不要新增索引。

#### 2.2.2 能用单索引，不用联合索引；能用窄索引，不用宽索引；能复用索引，不新建索引

回到线上案例，nc_tms_order、ct_order看看分别有哪些索引

> ct_order：13个组合索引 + 1个主键索引
>
> nc_tms_order：8个组合索引 + 1个主键索引

看到这里我开始凌乱，好像什么字段都可以加索引。

为此专门针对ct_order表两个具有比较性的索引做了性能测试

ct_order_code，lc_order_code区分度都是非常高的字段，前者是好于后者（联合station_id并没有起到太多优化作用）。

- idx_ct_order_code(ct_order_code),

- idx_ct_order_lc_order_code(station_id, lc_order_code)

### 3 最大选择性原则

#### 3.1 选择区分度高列做索引

  什么是区分度高的字段呢？

**一般两种情况不建议建索引：**

1. **一两千条甚至几百条，没必要建索引，让查询做全表扫描就好了**

   因为不是你建了就一定会走索引，执行计划会选择一个最优的方式，MySQL辅助索引的叶子节点并不直接存储实际数据，只是主建ID，再通过主键索引二次查找。这么一来全表可能很有可能效率更高。

2. **索引选择性较低的情况**

   所谓选择性（Selectivity），是指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值。

   ```
   Index Selectivity = Cardinality / #T
   ```

   显然选择性的取值范围为(0, 1]，选择性越高的索引价值越大，这是由B+Tree的性质决定的。

**回到线上案例 wh_id 最好不用做索引字段，这个和性别男、女作为索引字段没区别**

```sql
SELECT count(DISTINCT(wh_id)) / count(*) AS Selectivity
FROM `nc_tms_order_0340` `nc_tms_order`;
-- 0
```

选择性不足0.0001（精确值为0.00000666），按Selectivity值越大价值越大原则，实在没有什么必要为其单独建索引。

再看下distribution_order_id 单列索引。

```sql
SELECT count(DISTINCT(distribution_order_id)) / count(*) AS Selectivity FROM nc_tms_order_0340 nc_tms_order;
-- 0.0030
```

Selectivity = 0.0030 ，比之前有所优化，但其实不不是特别理想。

联合索引

```sql
SELECT count(DISTINCT(concat(wh_id,distribution_order_id)))/count(*) AS Selectivity FROM nc_tms_order_0340 nc_tms_order;
-- 0.0030
```

Selectivity = 0.0030，从值来看，这里建联合索引的价值并不是特别大。一个distrubution_id 搞定。



那么我们在建一个索引或联合索引的时候拿不准的时候可以先计算下选择性值以及通过explain测试。

一般情况，status、is_deleted列不建议建索引。

- 创建复合索引，需要注意把区分度最大的放到最前面。也就是值越大的放前面，当然需根据时间场景和sql通过执行计划进行优化。

- 前缀索引

  有一种与索引选择性有关的索引优化策略叫做前缀索引，就是用列的前缀代替整个列作为索引key，当前缀长度合适时，可以做到既使得前缀索引的选择性接近全列索引，同时因为索引key变短而减少了索引文件的大小和维护开销。 

### 4 其他

- 索引列不能参与计算

  比如`from_unixtime(create_time) = ’2017-11-11’`就不能使用到索引，语句应该写成`create_time = unix_timestamp(’2017-11-11’);`

- 主键最好使用自增型，保证数据连续性（mysql innodb 主键默认采用b+tree，索引和数据放在同一个btree中），不要使用uuid、hash、md5等

- 不要使用前匹配的like查询，会导致索引失效。可以使用后匹配like，如"xxx%"。

- 在字符串列上创建索引，尽量使用前缀索引。前缀基数根据具体业务，在匹配度和存储量（索引的存储量）之前做一个平衡。

- 不要使用 not inlike，会导致索引失效。not in可以用not exists替换。in和or所在列最好有索引

其实数据库索引调优，光靠理论是不行的，需要结合实际情况。MySQL机制复杂，如查询优化策略和各种引擎的实现差异等都会使情况变复杂。我们在了解这些原则和基础之上，要不断的实践和总结，从而真正达到高效使用MySQL索引的目的。

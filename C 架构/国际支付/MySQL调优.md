1、请DBA配合查找使数据库压力增加的SQL语句来源. 
2、查找到SQL语句问题原因是在WHERE条件后新增查询条件导致该条语句没有走索引. 
3、为新的WHERE条件增加新的组合索引并提交数据库变更. 
4、问题解决 



# 慢sql优化及MySQL索引原理

DB问题

- 一个是数据可靠性
- 一个是性能

## 1 线上案例

履行系统多级网络项目刚刚上线，为了支持多级作业，将运单和作业单做了分离，将原先的ct_order，拆分出nc_tms_order作业单。刚上线很多业务场景还没有加索引，刚上线几天随着业务不断深入，终于在今天爆发了。因为一个慢sql，导致DB服务器load上升，DB链接池耗尽，应用接口超时，甚至期间业务无法正常使用。

其中一个**sql**：

```sql
SELECT COUNT(1)
FROM`nc_tms_order_0340``nc_tms_order`
WHERE`nc_tms_order`.`wh_id`= 108884
AND`nc_tms_order`.`order_type`= 1
AND`nc_tms_order`.`distribution_order_id`='12059380340'
AND`nc_tms_order`.`is_deleted`= 0
```

从IDB上看到，平均响应时间5s，扫描303531行，流量一旦上来，基本就打满整个线程池，db性能急剧下降。

紧接着开发同学想到的是加索引，下午17.10分通过加上索引后，DB的各项性能数据恢复。

主要说说，索引应该如何加。DB性能指标图反映了从链接数耗尽、cpu接近100%，到恢复的过程。



如下图，新增

唯一索引：idx_whid_distributionorderid(wh_id, distribution_order_id) 

通过这个索引解决线上故障，接下来分析下优化过程。



IDB基本信息，第一眼看上去觉得索引有点多、而且乱。

> IDB基本信息
>
> 平均扫描行：303531
>
> 平均返回行：1
>
> 平均逻辑读：747528
>
> 唯一索引：wh_id：idx_whid_distributionorderid(wh_id, distribution_order_id) 
>
> 
>
> 索引明细：索引最左前缀
>
> - wh_id：唯一值数目：2、索引类型：BTREE
>
> - distribution_order_id：唯一值数目：2 K、索引类型：BTREE



wh_id，总条目数是30万+，wh_id唯一值条目为2，按照公式count(distinct col)/count(*)

区分度基本等于0。



sql解析的额外信息，当出现using index时，表示sql使用覆盖索引，性能较好，而当出现using filesort、using temporary、using where时，查询需要优化。



本次主要针对ct_order, tms_order等表进行优化，约影响索引20个。这里主要针对几个特例分析下。

### 1.1 出库单原先1650ms 缩减到10ms

```sql
SELECT COUNT(1) FROM nc_tms_order WHERE `STATUS` IN (?) AND `transfer_order_code` = ? AND .`wh_id` = ? AND `gmt_arrived` >= ? AND `is_deleted` = ?
```

- 原索引结构：idx(wh_id, site_id, transfer_order_code)

- 优化后：idx(transfer_order_code)

  很明显，原索引是一个无效索引，where条件中无site_id，按照索引最左前缀原则，那么原索引走到wh_id就停止了。

  新索引删除掉了wh_id, site_id字段后，成功优化。一直就讲过，wh_id县仓id在我们这种场景下作为索引字段是很业余的做法，先不说600个县仓通过取模#wh_id#%1024，落到相同分区表的数据很少外(据统计一般一个wh_id一个分区表，少数情况有两个县仓在同一分区表)，依据建索引的另一个原则，选择性最大的字段放在最前面，也就是说site_id应该放在wh_id前，transfer_order_code放在site_id前，最后的索引理论应该是Idx(transfer_order_code, site_id, wh_id). 好了，这样看下来好像没啥毛病。

  那么为什么我最终的索引是idx(transfer_order_code)呢，根据选择性最大原则，wh_id的区分度是可以忽略的，这个是个理由，其实还有一个更重要的原因是，根据我们这个sql的场景，通过确定transfer_order_codes的值，实际上就已经确定了wh_id, site_id；transfer_order_codes只可能属于一个wh_id、site_id，通过transfer_order_codes确定后面的wh_id,site_id索引条目只能是唯一的,再建索引非常冗余，有画蛇添足之意。打个比方，你在地图上搜索乐佳国际(前提默认在杭州市范围内)，你不需要冗余的说明“中国浙江省杭州市某某区某某路乐佳国际大厦”，你只会告诉搜索引擎“乐佳国际”四个字，它就会搜索你想要的大厦。我想说的是乐佳国际本身就是一个区分度很高的关键字了不需要你在多此一举告诉计算机这是在杭州余杭区的大厦（实际同城内大厦名称也不可能有重复的）。这让我又想起了国外人写信地址是从小到大，从经验上讲是符合索引字段排列顺序的，还是比较合理的。

  那么基于以上原因，我把所有不必要的wh_id、site_id都去掉了。当然我说的是不必要，那么情况下是必要的呢，比如说，我想要统计下某个站点下site_id的所有订单，那么这个时候，site_id作为索引字段是很必要。这个就好比，交易中，买家要查询自己所有已支付的订单，那么user_id即作为分区字段，同时也作为分区表的索引字段。

### 1.2 2000ms.领件缩减到20ms以内

```sql
SELECT COUNT(1) FROM `nc_tms_order_0609` `nc_tms_order`WHERE `nc_tms_order`.`STATUS` IN (3, 30)AND `nc_tms_order`.`SOP_TYPE` IN (1, 3, 5, 6)AND `nc_tms_order`.`wh_id` = 259681AND `nc_tms_order`.`dest_site_id` IN (266806)AND `nc_tms_order`.`gmt_arrived` >= '2017-08-14 00:00:00'AND `nc_tms_order`.`order_type` = 1AND `nc_tms_order`.`distribution_order_id` IS NULL AND `nc_tms_order`.`is_deleted` = 0
```

- 原索引：idx(wh_id, distribution_order_id)

- 优化后的索引：idx(distribution_order_id, status, dest_site_id)
  领件条件，distribution为空，唯一条目大概有6k，选择性不足，所以得加上status、dest_site_id，这个时候就能很快定位了。

### 1.3 8000ms 大包 缩减到20ms以内

```sql
select *
FROM ct_order `ct_order`WHERE `ct_order`.`STATUS` IN (?)AND `ct_order`.`IS_DELETED` = ?AND `ct_order`.`GMT_ARRIVED` >= ?AND `ct_order`.`GMT_ARRIVED` <= ?AND `ct_order`.`SOP_TYPE` IN (?)AND `ct_order`.`STATION_ID` = ?AND `ct_order`.`ORDER_TYPE` = ?AND `ct_order`.`DISTRIBUTION_ORDER_ID` IS NULLORDER BY `ct_order`.`ID` DESCLIMIT ?,?
```

-  原索引：idx(wh_id,distribution_order_id)

-  优化后索引：idx(distribution_order_id, sop_type)

 此索引与2索引其实是一个样的问题，为什么优化后的索引结构不同了，原因是应为根据这个sql场景，这里主要还是针对集包的领件，通过sop_type in (2) ，实际选择性很大，很快就能确定范围。这就是具体场景具体分析的案例。

## 2 建索引的几大原则

对于索引我总结三大原则：

1. 最左前缀原则；
2. 不冗余原则；
3. 最大选择性原则。

基本掌握这三条，对于索引的优化理论上是没有问题了。

### 最左前缀原则

一般在where条件中两个及以上字段时，我们会建联合索引。

高效使用索引的首要条件是知道什么样的查询会使用到索引，这个问题和B+Tree中的“最左前缀原理”有关，下面通过例子说明最左前缀原理。

MySQL中的索引可以以一定顺序引用多个列，这种索引叫做联合索引，一般的，一个联合索引是一个有序元组<a1,a2,a3...an>，其中各个元素均为数据表的一列，实际上要严格定义索引需要用到关系代数。另外，单列索引可以看成联合索引元素数为1的特例

#### 最左前缀是一个很重要的原则

MySQL会从左至右匹配，直到遇到范围查找(> < like between)就停止。

如：

select * from table1 where a=1 and b=2 and c<3 and d=9 ;

建立的联合索引为：(a,b,c,d)  实际使用的索引为(a,b,c)。因为遇到了c<3就停止了，d列就没有用上。

前面讲过联合索引是有序元组，

则MySQL实际建的索引为：(a) (a,b)  (a,b,c) (a,b,c,d)。

举个例子：where b=2 and c=3 and d=9 ；按照最左匹配原则，这个条件就没法走索引了，首先必须有a。

#### =,in可以乱序

查询优化器会帮你优化成索引可以识别的形式。也就是说，where b=2 and a=1 and c<3

使用的索引任然为(a,b,c)组合。



- **回到线上案例**：

索引:idx_whid_distributionorderid(wh_id,distribution_order_id)

索引组合 (wh_id) ,(wh_id,distribution_order_id)

相当于建了一个wh_id 的单列索引，也就是说当你要根据wh_id查询时，是不需要再新建索引了。



#### 不冗余原则

- **尽量扩展索引、不要新建索引**

MySQL目前主要索引有：FULLTEXT,HASH,BTREE

好的索引可以提高我们的查询效率，不好的索引不但不会起作用，反而给DB带来负担，基于BTREE结构，插入、修改都会重新调整索引结构，存储成本增加，写效率降低，同时DB系统也要消耗资源去维护。

基于刚才的最左匹配原则，尽量在原有基础上扩展索引，不要新增索引。



- **能用单索引，不用联合索引；能用窄索引，不用宽索引；能复用索引，不新建索引。**

回到线上案例：

nc_tms_order、ct_order看看分别有哪些索引

> ct_order
>
> - PRIMARY(id)
>
> - idx_sited_gmt_arried(sute_id, gmt_arrived)
> - idx_whid_ctordercode(wh_id, ct_order_code, site_id)
> - idx_whid_distributionorderid(**wh_id**, distribution_order_id)
> - idx_whid_mail_no(wh_id, site_id, mail_no)
> - idx_whid_siteid_order_type(wh_id, site_id, order_type, **status**)
> - idx_whid_tmsordercode(wh_id, site_id, **tms_order_code**)
> - idx_whid_id_site_id_transordercode(wh_id, site_id, transfer_order_code)
> - uk_tms_order_code(**tms_order_code**)
>
> 标黑点为需要优化点

ct_order的索引

> - PRIMARY(id)
> - idx_ct_distribution_order_id(station_id, distribution_order_id)
> - idx_ct_order_arrived_gmt(station_idm gmt_arrived)
> - idx_ct_order_code(ct_order_code)
> - idx_ct_order_deststation_id(station_id, dest_station_id, order_type)
> - idx_ct_order_driver_id(station_id, driver_id)
> - idx_ct_order_gmt_send(station_id, ic_order_id)
> - idx_ct_order_lc_order_code(station_id, lc_order_code)
> - idx_ct_order_mail_no(station_id, mail_no)
> - idx_ct_order_send(station_id, order_type, status, **is_deleted_options**, print_status)
> - idx_ct_order_status(station_id, **status**, order_type, dest_station_id)
> - idx_sort_uid_time(sort_out_uid, gmt_sort_out, dest_station_id)
> - idx_station_package_code(station_id, package_code)



看到这里我开始凌乱，好像什么字段都可以加索引。

为此专门针对ct_order表两个具有比较性的索引做了性能测试，ct_order_code，lc_order_code区分度都是非常高的字段，前者是好于后者（联合station_id并没有起到太多优化作用）。

idx_ct_order_code(ct_order_code),

idx_ct_order_lc_order_code(station_id, lc_order_code)



那么接下来我们说说那些字段适合建索引。

### 最大选择性原则

#### 选择区分度高列做索引

什么是区分度高的字段呢？

**一般两种情况不建议建索引：**

  **1、一两千条甚至几百条，没必要建索引，让查询做全表扫描就好了。**

因为不是你建了就一定会走索引，执行计划会选择一个最优的方式，msql辅助索引的叶子节点并不直接存储实际数据，只是主建ID，再通过主键索引二次查找。这么一来全表可能很有可能效率更高。

  **2、索引选择性较低的情况。**

所谓选择性（Selectivity），是指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值。


Index Selectivity = Cardinality / #T

显然选择性的取值范围为(0, 1]，选择性越高的索引价值越大，这是由B+Tree的性质决定的。



- **回到线上案例 wh_id 最好不用做索引字段，这个和性别男、女作为索引字段没区别：**

```sql
SELECT count(DISTINCT(wh_id))/count(*) AS Selectivity FROM `nc_tms_order`;
-- 0
```

选择性不足0.0001（精确值为0.00000666），按Selectivity值越大价值越大原则，实在没有什么必要为其单独建索引。

再看下distribution_order_id 单列索引。

```sql
SELECT count(DISTINCT(distribution_order_id))/count(*) AS Selectivity FROM nc_tms_order_0340 nc_tms_order;
-- 0.0030
```

Selectivity = 0.0030 ，比之前有所优化，但其实不不是特别理想。

联合索引

```sql
SELECT count(DISTINCT(concat(wh_id, distribution_order_id)))/count(*) AS Selectivity FROM nc_tms_order_0340 nc_tms_order;
-- 0.0030
```

Selectivity = 0.0030



从值来看，这里建联合索引的价值并不是特别大。一个distrubution_id 搞定。



那么我们在建一个索引或联合索引的时候拿不准的时候可以先计算下选择性值以及通过explain测试。

一般情况，status、is_deleted列不建议建索引。

- 创建复合索引，需要注意把区分度最大的放到最前面。也就是值越大的放前面，当然需根据时间场景和sql通过执行计划进行优化。

- 前缀索引

有一种与索引选择性有关的索引优化策略叫做前缀索引，就是用列的前缀代替整个列作为索引key，当前缀长度合适时，可以做到既使得前缀索引的选择性接近全列索引，同时因为索引key变短而减少了索引文件的大小和维护开销。

### **其他**

- 索引列不能参与计算

  比如from_unixtime(create_time) = ’2017-11-11’就不能使用到索引，语句应该写成create_time = unix_timestamp(’2017-11-11’);

- 主键最好使用自增型，保证数据连续性（MySQL innodb 主键默认采用b+tree，索引和数据放在同一个btree中），不要使用uuid、hash、md5等

- 不要使用前匹配的like查询，会导致索引失效。可以使用后匹配like，如"xxx%"。

- 在字符串列上创建索引，尽量使用前缀索引。前缀基数根据具体业务，在匹配度和存储量（索引的存储量）之前做一个平衡。

- 不要使用 not inlike，会导致索引失效。not in可以用not exists替换。in和or所在列最好有索引



说了这么多留action吧，大家回去看看ct_order,nc_tms_order看看如何优化吧。



其实数据库索引调优，光靠理论是不行的，需要结合实际情况。MySQL机制复杂，如查询优化策略和各种引擎的实现差异等都会使情况变复杂。我们在了解这些原则和基础之上，要不断的实践和总结，从而真正达到高效使用MySQL索引的目的。



### 执行计划explain命令

**explain 是sql优化神奇。**

以下均为工具介绍，可留做备查。

**explain用法**

EXPLAIN tbl_name或：EXPLAIN [EXTENDED] SELECT select_options

举例

1. ```
   MySQL> explain select * from event;  
   +—-+————-+——-+——+—————+——+———+——+——+——-+  
   | id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |  
   +—-+————-+——-+——+—————+——+———+——+——+——-+  
   | 1 | SIMPLE | event | ALL | NULL | NULL | NULL | NULL | 13 | |  
   +—-+————-+——-+——+—————+——+———+——+——+——-+  
   1 row in set (0.00 sec)
   ```

**各个属性的含义**

- **id：**select查询的序列号

- **select_type：**select查询的类型，主要是区别普通查询和联合查询、子查询之类的复杂查询。

- **table**：输出的行所引用的表。

- **type**：联合查询所使用的类型。

type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref。

就type进行详细的介绍：**all :** 即全表扫描

**index :** 按索引次序扫描，就type进行详细的介绍：

System,const,eq_ref,ref,range,index,all

**all :** 即全表扫描



**index :** 按索引次序扫描，先读索引，再读实际的行，结果还是全表扫描，主要优点是避免了排序。因为索引是排好的。

**range:**以范围的形式扫描。

explain select * from a where a_id > 1\G

**ref:**非唯一索引访问(只有普通索引)

create table a(a_id int not null, key(a_id));

insert into a values(1),(2),(3),(4),(5),(6),(7),(8),(9),(10);

mysql> explain select * from a where a_id=1\G

**eq_ref:**使用唯一索引查找(主键或唯一索引)

**const:**常量查询

当出现using index时，表示sql使用覆盖索引，性能较好，而当出现using filesort、using temporary、using where时，查询需要优化。

先读索引，再读实际的行，结果还是全表扫描，主要优点是避免了排序。因为索引是排好的。

**range:**以范围的形式扫描。

explain select * from a where a_id > 1\G

**ref:**非唯一索引访问(只有普通索引)

create table a(a_id int not null, key(a_id));

insert into a values(1),(2),(3),(4),(5),(6),(7),(8),(9),(10);

mysql> explain select * from a where a_id=1\G

**eq_ref:**使用唯一索引查找(主键或唯一索引)

**const:**常量查询

当出现using index时，表示sql使用覆盖索引，性能较好，而当出现using filesort、using temporary、using where时，查询需要优化。

**possible_keys**：指出MySQL能使用哪个索引在该表中找到行。如果是空的，没有相关的索引。这时要提高性能，可通过检验WHERE子句，看是否引用某些字段，或者检查字段不是适合索引。

- **key**：显示MySQL实际决定使用的键。如果没有索引被选择，键是NULL。

- key_len：显示MySQL决定使用的键长度。如果键是NULL，长度就是NULL。文档提示特别注意这个值可以得出一个多重主键里MySQL实际使用了哪一部分。

- ref：显示哪个字段或常数与key一起被使用。

- rows：这个数表示MySQL要遍历多少数据才能找到，在innodb上是不准确的。

- Extra：如果是Only index，这意味着信息只用索引树中的信息检索出的，这比扫描整个表要快。

如果是where used，就是使用上了where限制。

如果是impossible where 表示用不着where，一般就是没查出来啥。

如果此信息显示Using filesort或者Using temporary的话会很吃力，WHERE和ORDER BY的索引经常无法兼顾，如果按照WHERE来确定索引，那么在ORDER BY时，就必然会引起Using filesort，这就要看是先过滤再排序划算，还是先排序再过滤划算。
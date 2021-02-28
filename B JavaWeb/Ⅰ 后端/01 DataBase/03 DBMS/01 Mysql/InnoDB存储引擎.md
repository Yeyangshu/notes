# InnoDB 存储引擎

InnoDB 存储引擎特点：

- 行锁设计
- 支持 MVCC
- 支持外键
- 提供一致性非锁定读
- 最有效地利用以及使用内存和 CPU

使用 MVCC 来支持高并发，通过 Next-Key Locking（间隙所）策略避免幻读
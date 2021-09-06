## undo log

想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行回滚。而在 MySQL 中，恢复机制是通过回滚日志（undo log）实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后才对数据库中的数据执行修改操作。

回滚日志会先于数据持久化到磁盘上。这样就保证了即使遇到数据库突然宕机等情况，当用户再次启动数据库的时候，数据库还能够通过查询回滚日志来回滚将之前未完成的事务。

## redo log

redo log（重做日志）是 InnoDB 存储引擎独有的，它让 MySQL 拥有了崩溃恢复能力。

InnoDB 作为 MySQL 的存储引擎，数据是存放在磁盘中的，但如果每次读写数据都需要磁盘 IO，效率会很低。为此，InnoDB 提供了缓存(Buffer Pool)，Buffer Pool 中包含了磁盘中部分数据页的映射，作为访问数据库的缓冲：
当从数据库读取数据时，会首先从 Buffer Pool 中读取，如果 Buffer Pool 中没有，则从磁盘读取后放入 Buffer Pool；当向数据库写入数据时，会首先写入 Buffer Pool，Buffer Pool 中修改的数据会定期刷新到磁盘中（这一过程称为刷脏）。

Buffer Pool 的使用大大提高了读写数据的效率，但是也带了新的问题：如果 MySQL 宕机，而此时 Buffer Pool 中修改的数据还没有刷新到磁盘，就会导致数据的丢失，事务的持久性无法保证。

于是，redo log 被引入来解决这个问题：当数据修改时，除了修改 Buffer Pool 中的数据，还会在 redo log 记录这次操作；当事务提交时，会调用 fsync 接口对 redo log 进行刷盘。如果 MySQL 宕机，重启时可以读取 redo log 中的数据，对数据库进行恢复。
redo log 采用的是 WAL（Write-ahead logging，预写式日志），所有修改先写入日志，再更新到 Buffer Pool，保证了数据不会因 MySQL 宕机而丢失，从而满足了持久性要求。

![](../../images/20210906-1.png)

引入 Buffer Pool 是为了减少磁盘读写，提高读写效率。引入 redo log 是为了避免宕机时 Buffer Pool 的数据丢失。记录 redo log 也是对磁盘的写入为，何引入不会造成效率降低，或者说为什么它比直接将 Buffer Pool 中修改的数据写入磁盘(即刷脏)要快呢？
主要有以下两方面的原因：

* 刷脏是随机 IO，因为每次修改的数据位置随机，但写 redo log 是追加操作，属于顺序 IO；
* 刷脏是以数据页（Page）为单位的，MySQL 默认页大小是 16KB，一个 Page 上一个小修改都要整页写入；而 redo log 中只包含真正需要写入的部分，无效 IO 大大减少。

## redo log 与 binlog 对比

* 作用不同：redo log 是用于 crash recovery 的，保证 MySQ L宕机也不会影响持久性；binlog 是用于 point-in-time recovery 的，保证服务器可以基于时间点恢复数据，此外 binlog 还用于主从复制;
* 层次不同：redo log 是 InnoDB 存储引擎实现的，而 binlog 是 MySQL 的服务器层实现的，同时支持 InnoDB 和其他存储引擎；
* 内容不同：redo log 是物理日志，内容基于磁盘的 Page；binlog 的内容是二进制的，根据 binlog_format 参数的不同，可能基于 sql 语句、基于数据本身或者二者的混合。
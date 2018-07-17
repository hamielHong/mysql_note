mysql锁总结
===

读写锁
---

处理并发读写问题的一个解决方案是实现一个由两种锁组成的系统：
| 锁一 | 锁二 |
|---|---|
| 共享锁(shared lock) | 排它锁(exclusive lock) |
| 读锁(read lock) | 写锁(write lock) |

读锁是共享的、互不阻塞的；

写锁是排他的，一个写锁会阻塞其他的写锁和读锁。

这能确保在同一时间只有一个用户能执行写入，并防止其他用户读取正在写入的同一资源。

锁粒度
---

锁定的粒度越小，系统的并发性越高。
锁的数量越多，系统的开销越大。

所谓的锁策略，就是在锁的开销和数据的安全性之间寻求平衡。

1. 表锁(table lock)

    表锁是mysql中最基本的锁策略，并且是开销最小的策略，它会锁定整张表。

    在特定的场景中，表锁也可能拥有良好的性能。例如，READ LOCAL表锁支持某些类型的 **并发写** 操作。另外，写锁比读锁有更高的优先级，一个写锁的请求可能会被插入到读锁队列的前面。

    mysql本身会使用各种有效的表锁来实现不同的目的。例如，服务器会为诸如 ALERT TABLE 之类的语句使用表锁，而忽略存储引擎的锁机制。

2. 行级锁

    行级锁可以最大程度的支持并发处理（同时也带来了最大的锁开销）。行级锁只在存储引擎层实现，而mysql服务器层没有实现。
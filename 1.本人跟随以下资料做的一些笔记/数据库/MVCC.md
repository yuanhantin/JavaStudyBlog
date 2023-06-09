# MVCC

##  版本链

我们前边说过，对于使用 InnoDB 存储引擎的表来说，它的聚簇索引记录中都包含两个必要的隐藏列 

- trx_id ：每次一个事务对某条聚簇索引记录进行改动时，都会把该事务的 事务id 赋值给 trx_id 隐藏列。

- roll_pointer ：每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到 undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

  假设之后两个 事务id 分别为 100 、 200 的事务对这条记录进行 UPDATE 操作，操作流程如下：

  ![image-20230619133617294](https://s2.loli.net/2023/06/19/iO5ZEMjohWn7XvV.png)

每次对记录进行改动，都会记录一条undo日志 ，每条 undo日志 也都有一个 roll_pointer 属性（ INSERT 操作 对应的 undo日志 没有该属性，因为该记录并没有更早的版本），可以将这些 undo日志 都连起来，串成一个链表，所以现在的情况就像下图一样：

![image-20230619132714830](https://s2.loli.net/2023/06/19/ZrUWAN9cb3BRp5u.png)

对该记录每次更新后，都会将旧值放到一条 undo日志 中，就算是该记录的一个旧版本，随着更新次数的增多， 所有的版本都会被 roll_pointer 属性连接成一个链表，我们把这个链表称之为 版本链 ，版本链的头节点就是当 前记录最新的值。另外，每个版本中还包含生成该版本时对应的 事务id ，这个信息很重要，我们稍后就会用到。

## ReadView

我们来引出一下readview是什么东西：

对于使用 READ UNCOMMITTED 隔离级别的事务来说，由于可以读到未提交事务修改过的记录，所以直接读取记录的最新版本就好了

对于使用 SERIALIZABLE 隔离级别的事务来说，则是使用加锁的方式来问记录

但是！！！对于使用 READ COMMITTED 和 REPEATABLE READ 隔离级别的事务来说

都**必须保证读到已经提交了的事务**修改过的记录，也就是说假如另一个事务已经修改了记录但是尚未提交， 是不能直接读取最新版本的记录的，**核心问题就是：需要判断一下版本链中的哪个版本是当前事务可见的。**为此，设计 InnoDB 的大叔提出了一个 ReadView 的概念，这个 ReadView 中主要包含4个比较重要的内容：

- m_ids ：表示在生成 ReadView 时当前系统中活跃的读写事务的事务id 列表。 
- min_trx_id ：表示在生成 ReadView 时当前系统中活跃的读写事务中最小的事务id ，也就是 m_ids 中的最小值。 
- max_trx_id ：表示生成 ReadView 时系统中应该分配给下一个事务的 id 值。 
  - 小贴士： 注意max_trx_id并不是m_ids中的最大值，事务id是递增分配的。比方说现在有id为1，2，3这三 个事务，之后id为3的事务提交了。那么一个新的读事务在生成ReadView时，m_ids就包括1和2，mi n_trx_id的值就是1，max_trx_id的值就是4。 
- creator_trx_id ：表示生成该 ReadView 的事务的事务id 。 
  - 小贴士： 我们前边说过，只有在对表中的记录做改动时（执行INSERT、DELETE、UPDATE这些语句时）才会 为事务分配事务id，否则在一个只读事务中的事务id值都默认为0。

有了 ReadView ，这样在访问某条记录时，只需要按照下边4个步骤判断记录的某个版本是否可见：

- 如果被访问版本的 trx_id = creator_trx_id ，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。 
- 如果被访问版本的 trx_id < min_trx_id ，表明生成该版本的事务在当前事务生成 ReadView 前已经提交，所以该版本可以被当前事务访问。 
- 如果被访问版本的 trx_id >  max_trx_id ，表明生成该版本的事务在当前事务生 成 ReadView 后才开启，所以该版本不可以被当前事务访问。 
- 如果被访问版本的 min_trx_id <  trx_id  < max_trx_id ，那就需要判断一下 trx_id 属性值是不是在 m_ids 活跃事务列表中。
  - 如果在，说明创建 ReadView 时生成该版本的事务还是活跃的，还没提交，该版本不可以被访问。
  - 如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。

## ReadView生成时机

读已提交 和 可重复读 隔离级别最大的区别就是它们生成ReadView的时机不同：

- READ COMMITTED —— 每次读取数据前都生成一个ReadView
  - 每次select操作都生成一个ReadView，所以每一次查询都在用一个最新生成的ReadView进行上述4步走，所以查出来的应该是最新提交的事务中的最后一个操作（因为一个事务可能有多个增改操作）
- REPEATABLE READ —— 在第一次读取数据时生成一个ReadView
  - 意味着你在第一次执行select语句之后，不管增删改了多少次，永远拿第一次select生成的ReadView来进行刚刚说的4个步骤判断，所以一直查询的是第一次select语句所查出来的东西。

具体例子这里不过多赘述，还是自行看书。
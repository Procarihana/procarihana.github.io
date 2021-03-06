---
title: "MYSQL"
date: 2020-09-08T11:24:13+08:00
draft: false
tags: ["MYSQL","InnDB","MyISAM","Mysql事务"]
---

## 数据库主键
- UUID是无序的，所以不能用UUID进行数据库主键查找

```
主键 -- 唯一标识一条记录，不能有重复的，不允许为空
外键 -- 表的外键不一定是另一表的主键，但必须是唯一性索引,外键可以有重复的, 可以是空值
索引 -- 该字段没有重复值，但可以有一个空值
作用：
主键 -- 用来保证数据完整性
外键 -- 用来和其他表建立联系用的
索引 -- 是提高查询排序的速度  
个数：
主键 -- 主键只能有一个
外键 -- 一个表可以有多个外键
索引 -- 一个表可以有多个唯一索引
```
## SQL标准事务
- 原子性：事物内的业务全部完成才算成功，有一步失败都会回滚到最初。
- 一致性：满足数据库完整性的约束，就是建表的时候数据的类型约束、数据范围等，都会满足。
- 隔离性：写的内容没有提交前，其他读的对象是只能够执行没有改变前的数据内容，只有写的内容提交后才能够读到变更后的内容。
- 持久性：事物一旦提交，就会永久保存到数据库里面，即使系统发生故障，数据库也能够将数据恢复。
#### 隔离级别
- read-commited 云服务器一般都会用这个，因为隔离级别比较高
- Console `begin`并不是开始，只有执行语句才是真正的开始
- `read uncommitted` 读未提交： 一个事务还没提交时，它做的变更就能被别的事务看到。
>- 脏读： 没有commit 的数据被 另一个console读取到了 
>- 可以获得最新的数据，
- 读提交（READ COMMITTED）只能看到提交的事物，但是不可重复度，因为执行多次的查询可能会得到不同的结果（因为多次查询中间如果其他事务进行了commit，查询的结果就会出现不同）
- 可重复读：在同一个事务中，多次进行查询的到的结果也是一样的，即使在多次读取的中间有其他事务进行了commit，读取的结果也不会因为其他的commit而改变。(没有commit也不能够读取到)
> - 但是可能会出现幻读：在其他事务已经插入了某个ID的数据且commit，在可重复读的事务里不能够查询到这个数据，还能够用同样的ID进行数据的插入和查询，没有出现主键冲突的现象，commit后查询能够看到同一个ID插入了两条不一样的数据
> - - 但是在mysql 8 后就禁用了，一般都不会出现
>- 在一个事务中SELECT操作一致，就是依靠ReadView(一致性视图的生成)
- 可串行化（SERIAIZABLE）：
- - 事务查询数据时，如果其他事务在进行数据的变更时候没有commit，就必须等待直到其他事务commit后才能够查询出数据结果，否则会一直等待到超时
- - 对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。
- - 实现`serializable`(可串行)采用加锁的策略通过牺牲并发能力而保证数据安全)
- 事务隔离级别就是系统并发能力和数据安全性间的妥协，隔离性越高，数据库的性能就越差
## Mysql 事务
- 在不同的隔离级别下，数据库通过 MVCC 和隔离级别，让事务之间并行操作遵循了某种规则，来保证单个事务内前后数据的一致性
- Multiversion concurrency control 多版本并发控制
- - 并发访问（读或者写）数据库时，对正在事务内处理的数据做多版本的管理，用来避免由于写操作的堵塞，而引发读操作失败的并发问题。
- - MVCC机制下基于生成的Undo log链和一致性视图ReadView来实现的
####  MYISAM并不支持事务
- 独立于操作系统，可以轻松地将其从Windows服务器移植到LIinux 里面
- 选择密集型的表。 MyISAM存储引擎在筛选大量数据时非常迅速，这是它最突出的优点。
- 插入密集型的表。 MyISAM的并发插入特性允许同时选择和插入数据。
#### InnoDB实现了MVCC的事务并发处理机
- 需要事务支持，并且有较高的并发读取频率选择InnDB
>- 单纯的MVCC机制并不能解决幻读问题,InnoDB也是通过加间隙锁来防止幻读
- InnoDB 下的 MVCC 实现原理（Undo log）
>- 在InnoDB中MVCC的实现通过两个重要的字段进行连接：`DB_TRX_ID`和`DB_ROLL_PT`，在多个事务并行操作某行数据的情况下，不同事务对该行数据的UPDATE会产生多个版本，数据库通过DB_TRX_ID来标记版本，然后用DB_ROLL_PT回滚指针将这些版本以先后顺序连接成一条 `Undo Log` 链。
>1. `DB_TRX_ID`: 事务id，6byte，每处理一个事务，值自动加一。
>- - InnoDB中每个事务有一个唯一的事务ID叫做 transaction id。在事务开始时向InnoDB事务系统申请得到，是按申请顺序严格递增的
>- - 每行数据是有多个版本的，每次事务更新数据时都会生成一个新的数据版本，并且把transaction id赋值给这个数据行的`DB_TRX_ID`
>2. `DB_ROLL_PT`: 回滚指针，7byte
>- - 指向当前记录的`ROLLBACK SEGMENT` 的undolog记录，通过这个指针获得之前版本的数据。该行记录上所有旧版本在 `undolog `中都通过链表的形式组织。
>3. 还有一个DB_ROW_ID(隐含id,6byte，由innodb自动产生)，
- - InnoDB下聚簇索引B+Tree的构造规则:  
如果声明了主键，InnoDB以用户指定的主键构建B+Tree，如果未声明主键，InnoDB 会自动生成一个隐藏主键，说的就是DB_ROW_ID。另外，每条记录的头信息（record header）里都有一个专门的bit（deleted_flag）来表示当前记录是否已经被删除
#### MVCC-- 一致性视图的生成（ReadView）
- RC（readcommitted） 在A事务提交后，其他事务可见
- RR（repeatable read） A事务中SELECT操作一直，依靠的就是READVIEW、
- 读不提交 直接能够读取最新数据
- 可串行 通过加锁来保证数据安全
--> `RC``RR`需要MVCC来保证ReadView
>- Read committed 
>- - readview 会在事务中的每一个SELECT语句查询发送前生成（也可以在声明事务时显式声明`START TRANSACTION WITH CONSISTENT SNAPSHOT`），因此每次SELECT都可以获取到当前已提交事务和自己修改的最新版本。  
>--> 每一次SELECT 查询都能够获得最新的其他事务已经commit 的版本

>- Repeatable read 
>- - 每个事务只会在第一个SELECT语句查询发送前或显式声明处生成，其他查询操作都会基于这个ReadView，这样就保证了一个事务中的多次查询结果都是相同的   
>-->  只有第一次SELECT的时候会获得最近的commit 后的数据，之后的操作都会基于这个版本进行

- InnDB 为每一事务构造了一个数组

## Mysql 为什么用B+tree 
- MySQL 跟 B+ 树没有直接的关系，真正与 B+ 树有关系的是 MySQL 的默认存储引擎 InnoDB，MySQL 中存储引擎的主要作用是负责数据的存储和提取，除了 InnoDB 之外，MySQL 中也支持 MyISAM 作为表的底层存储引擎。
- 因为B+ tree 已经对数据进行过排序，在使用索引的时候就能够直接使用，不需要重新排序产生排序文件。

1. hash值：存储是无序的，所以无法使用范围查找、排序等。
2. 平衡二叉树：左字数和右字数的高度绝对值不会超大于一，且排列有序。
缺点：树越高查找数越慢。且数据的查找是中序根查找，出现回旋查找的情况，导致查找时间变慢。  
但是每个节点只能够存放一个数据，导致数的高度会增加，随着高度增加，查找数据的速度也会下降。在查找数据的时候，会有回旋查找情况发生，
3. B tree ： 平衡二叉树的基础上，一个节点能够存两个值，使得树的高度变小，从而使数据的查找速度变快，但是仍然是中序根查询，仍然会有回旋查找的问题
4. B+ tree：
>1. 在B tree 的基础上，叶子节点有箭头，把所有的树叶子都进行链表排序。
>2. 非叶子节点的数都会在叶子节点里面进行排序。
>3. 但是非叶子节点只存储key，而叶子节点即存储key也存储value。(key就是数据的顺序，value就是数据值的位置。)
>4. 值得注意的是第一个和最后一个叶子节点即是叶子节点也是非叶子节点
## 索引失效
- 多个键值的B+数
-  联合索引：由两个字段（a,b）完成，先通过a字段排序，完成后再进行b字段进行排序，是有优先顺序的（遵循最佳左前序法则）
- 使用字母进行范围查找的时候是26个字母的排行顺序进行排序
- 只有a使用`=`，b进行搜索才能够使用索引
>- a 是有序的，相同a 的b 是有序的，而不同a的b是无序的。
>- 如果没有了a ，b就会无序 --> 没有a情况下用寻找b就是在无序的B+树上寻找这个值b，不能够使用索引的方法，只能够使用全表扫描的方法，使得索引失效
- 范围查找
>- a使用范围查找的时候，找出来的数据虽然a 是有序的，但是b是没有序的，所以当查找b 的时候不能够通过二分查找完成索引
- - like 失效查询： 百分号放左边或者两边都是不会使用索引的，只有百分号放在右边的某些情况才会使用到索引
- - - 就是只有使用前缀查找的情况下，a才是有顺序的，如果使用中缀或者后缀a就没有顺序，最左前序法则排列的B+数的中序后序是没有顺序的，因此在无序的情况下查找是使用不到索引的
>- like 匹配/模糊匹配
>- like 匹配/模糊匹配，会与` %` 和 `_ `结合使用。
>- - '%a'     //以a结尾的数据 --> 前缀 abb
>- - 'a%'     //以a开头的数据 --> 后缀 bsa
>- - '%a%'    //含有a的数据 --> 中缀
>- - '_a_'    //三位且中间字母是a的
>- - '_a'     //两位且结尾字母是a的
>- - 'a_'     //两位且开头字母是a的
>- `%`：表示任意 0 个或多个字符。可匹配任意类型和长度的字符，有些情况下若是`中文`，请使用两个百分号（%%）表示。
>- `_`：表示任意单个字符。匹配单个任意字符，它常用来限制表达式的字符长度语句。
>- `[]`：表示括号内所列字符中的一个（类似正则表达式）。指定一个字符、字符串或范围，要求所匹配对象为它们中的任一个。
>- `[^]` ：表示不在括号所列之内的单个字符。其取值和 `[] `相同，但它要求所匹配对象为指定字符以外的任一个字符。
>- 查询内容包含通配符时,由于通配符的缘故，导致我们查询特殊字符 `“%”、“_”、“[” `的语句无法正常实现，而把特殊字符用` “[ ]” `括起便可正常查询。

## CSV
- CSV 存储引擎是基于 CSV 格式文件存储数据。（，，，）

- CSV 存储引擎因为自身文件格式的原因，所有列必须强制指定 NOT NULL 。
- CSV 引擎也不支持索引，不支持分区。
- CSV 存储引擎也会包含一个存储表结构的 .frm 文件，还会创建一个 .csv 存储数据的文件，还会创建一个同名的元信息文件，该文件的扩展名为 .CSM ，用来保存表的状态及表中保存的数据量。
每个数据行占用一个文本行。

因为 csv 文件本身就可以被Office等软件直接编辑，保不齐就有不按规则出牌的情况，如果出现csv 文件中的内容损坏了的情况，也可以使用 CHECK TABLE 或者 REPAIR TABLE 命令检查和修复

## MYSQL 数据类型
- TINYINT （3个字符）
- - TINYINT（2）、（3）还是保留TINYINT属性，在java中用Integer 配置
- TINYINT(1) （1个字符）--自动转化成--> BIT（一个字符）
- nvarchar(n) 用于存储中文 韩文等。字节存储大小是所输入字符个数的两倍
- decimal(x,y) --> x为数值的总长度，y为精确到的小数位置

# 索引 
- 帮助Mysql 高效获取数据的数据结构
- 实现高级查找的`算法`的数据结构
-  索引一般以文件形式储存在磁盘上
---
数据库结构
- xxx.frm 数据结构
- xxx.idb index 索引，需要通过数据库的序列化和返序列化完成读取，直接打开是不行的
- xxx.MYD 数据
- xxx.MYI InnoDB
--- 
磁盘存取原理
- 程序运行数据储存在内存里面，mysql 在内核里面调取数据，把数据拷贝到用户内存里面
- io 寻找 先寻道，找到数据所储存在的磁道位置，找到后旋转到相应的地方，预读完成数据从磁盘中读取（4K*N） 4k是计算机里面的一页。计算机的局部性原理--> 读到地方的附近都会有可能被读取，通过预测读取能够减少耗时
- 但是通过这种方式寻找，数据量越大，寻找的时间就越长，就需要算法来提升
- 一次预读内存为16K
---
- 红黑树
1. 约束逻辑
- 节点是红色或者是黑色
- 根节点是黑色
- 每个红色节点的两个子节点都是黑色
- 新插入的节点默认是红色
2. 变化
- 变色
- 旋转
3. 缺点
- 红黑树的高度高
- 红黑树每次一插入数据都需要进行旋转等
---
- BTree
1. 逻辑约束
- 度（Degfree）一节点的数据储存个数
- 叶节点具有相同的深度
- 叶节点的指针为空
- 节点中的数据key从左到右递增排列
2. 缺点
- 度和数据一同存储，data占据的内存越多，度的个数就会越少。也就是数据量越大，储存的个数就少
- 范围查询，如果数据没有的话就要返回根节点查找
---
- B+tree
- 非叶子节点不储存data，只存储key，可以`增大度`
- 所有叶子节点都有一个链指针，在范围查找的时候就不需要回去根节点查询下一页的内容，而能够直接通过链指针跳转到下一页继续完成查找
- 顺序访问指针，提高区间访问的性能
---
Innodb
- header 记录上一页和下一页的指针
- 用户记录 数据
- 页目录 目录记录数据的一个范围，能够在数据查找的时候快速查找出一个较小的范围，提高速度
--- 
索引

---
哈希
- 数组特性：连续储存空间
- 哈希索引：数组加链表
- 通过函数算出一个数据的地址
- 但是不能够保证绝对的唯一，这时候就需要链表来保存相同的哈希地址的不同的数据
- 数据查找的时候只需要算出哈希值就能够找到该数据
>- 键值唯一，哈希索引明显有绝对优势
>- 无法完成范围查询检索
>- 无法利用索引完成排序，以及like ‘xxx%’ 这样的部分模糊
>- 不支持多列联合索引的
>- 因为哈希碰撞问题，索引效率是极低的
- 哈希值是有范围，但是数据经过哈希后就没有范围了，所以并不能运用哈希范围查找
- 模糊查询哈希后是没有意义的，且多联合索引也是同样的道理
- 但是每次单次查询的话，因为哈希查询是O(1)所以还是有优势的
---
联合索引
- 表里面经常要用到的字段就使用索引（栏位）
- 查询出来的数据从左到右从小到大排序
- type 可见是否有用到索引
- 查询优化：如果查询的字段都有用到索引，而查询的语句顺序非索引顺序的话，还是会因为查询优化进行顺序调整后进行联合查询
- 范围查询也有使用索引
- 使用`!=`没有用索引
 
-----
1. 聚簇索引和非聚簇索引的区别
- 数据和索引结构是否一起
- 主键索引就是一聚zu，根据主键能够查找到数据
- 非聚zu索引（非主键索引），通过索引找到数据的主键，找到后就要回到主键索引里面才能够获得所有的数据
2. InnoDB怎么保证必有主键
- 如果建表时没有建立主键，就会通过唯一键来定义为主键，如果两个都没有就会自动生成一个隐藏的主键
3. 为什么推荐``使用整型的自增主键`
- uuid 作为主键 int 8个字节就能够存储
- 数据从左到右从小到大排列储存，用整型进行比较更快捷，且自增的话就能保证顺序，而如果用字符ID的话就可能出现需要数据的重新排序，如果涉及不同页的整理就需要整个索引结构的重建
4. 为什么非主键索引结构叶子节点储存的是`主键值而不是数据`
- 数据一致性 不同的索引结构都存储一份，改动的成本非常高，而通过非主键索引就能够共用一份主键索引，从而降低改动成本
- 存储空间 非主键能够减少数复制的空间成本，达到时间换空间
5. 数据库语言类型
- 数据查询语言（DQL）：是由SELECT子句，FROM子句，WHERE子句组成的查询块
- 数据操纵语言（DML）: SELECT(查询) INSERT(插入) UPDATE(更新) DELETE(删除）
- 数据定义语言（DDL）：CREATE(创建数据库或表或索引）ALTER(修改表或者数据库）DROP(删除表或索引）
- 数据控制语言（DCL）：GRANT(赋予用户权限） REVOKE(收回权限） DENY(禁止权限)
- 事务控制语言（TCL）：SAVEPOINT (设置保存点）ROLLBACK (回滚) COMMIT(提交)

## pratice
- https://blog.csdn.net/weixin_43930865/article/details/89525700

## 修改用户
https://www.cnblogs.com/zhangjianqiang/p/10019809.html

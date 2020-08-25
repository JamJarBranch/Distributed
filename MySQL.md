# `MySQL`



前端的web服务器或者后端的数据库服务器 承受巨大的压力

web服务器可能简单的进行 横向的数量的扩展，

但数据库服务器 因为`完整性和一致性` 不能进行横向扩展



数据库架构： 一主多从

数据库集群的监控信息 来查看什么影响了数据库的性能



影响数据库性能的因素

- 服务器硬件
  - 磁盘I/O ( 网卡流量是另外一种磁盘资源 )
  - CPU
  - 内存
- 服务器系统
- 数据库存储引擎的选择( 插件式引擎 )
  - MyISAM： 不支持事务，表级锁
  - InnoDB：事务级存储引擎，完美支持行级锁，事务ACID特性
- 数据库参数配置
- 数据库结构设计和SQL语句 ( `sql`查询速度 )
- 大表
- 大事务



超高的`QPS`和`TPS`

> `QPS` 每秒处理的查询量
>
> `TPS` 

- 带来的风险：效率地下的`SQL`

`MySQL`中并不支持 多CPU的并发运算。即每一个sql 只能用到一个CPU



大量的并发和超高的CPU使用率

- 风险
  - 大量的并发： 数据库连接数被占满(max_connections默认为100)
  - 超高的CPU使用率： 因CPU资源耗尽而出现宕机



磁盘I/O

- 风险
  - 磁盘I/O性能突然下降
    - 解决1：使用更快的磁盘设备
    - 解决2：调整 其他大量消耗磁盘性能的计划任务，做好磁盘维护



网卡流量

- 风险
  - 网卡IO被占满(1000Mb/8 ~= 100MB)
  - 避免无法连接数据库的情况
    - 减少从服务器的数量
    - 进行分级缓存
    - 避免使用 `select *` 来进行查询
    - 分离业务网络和服务器网络【避免主从同步，网络备份的影响网络性能】



大表

> - 记录行数巨大，单表超过 千万行
> - 表数据文件巨大，大小超过10 G

大表对查询的影响

- 慢查询： 很难在一定事件内过滤出需要的数据

大表对DDL操作的影响

- 建立索引需要很长的时间
  - 风险
    - <5.5 建立索引会锁表
    - `>=`5.5 不会锁表但会造成 主从的延迟
- 修改表结构会进行长时间的锁表
  - 风险
    - 长时间的主从延迟
    - 影响正常的数据操作
      - 长时间的锁表会造成数据库连接数被占满，导致前台服务器会造成 500 错误

处理数据库中的大表

- 分库分表 将一张大表分成多个小表
  - 难点
    - 分表主键的选择
    - 分表后 跨分区数据的查询和统计
- 大表的历史数据归档 减少对前后端业务的影响 
  - 难点
    - 归档时间点的选择
    - 如何进行归档操作，归档要把归档的数据从现表进行删除



大事务

> 什么是事务？
>
> 事务是数据库系统区别于其他一切文件系统的重要特性之一
>
> - 数据库突然奔溃后仍然可以恢复 数据库中的数据，使数据保持一致性
>
> 事务是一组具有原子性的SQL语句，或是一个独立的工作单元
>
> 事务满足
>
> - 原子性 ATOMICITY
>   - 要么全部完成，要么全部失败混滚
> - 一致性 CONSISTENCY
>   - 从一种一致性状态 转换到 另外一种一致性状态。在事务开始前和结束后数据的完整性没有被破坏
> - 隔离性 ISOLATION
>   - 一个事务对数据库中数据的修改在未提交前对 其他事务是不可见的
>   - 四种隔离级别【隔离性由低到高；并发性由高到低】
>     - 未提交读
>     - 已提交读【不可重复读】
>     - 可重复读
>     - 可串行化
> - 持久性 DURABILITY 
>   - 事务提交后所做的修改永久保存在数据库中，系统奔溃提交的修改书也不会丢失
>
> 什么是大事务？
>
> 运行时间比较长，操作的数据比较多的事务

- 风险
  - 锁定太多的数据，造成大量的阻塞和锁超时
  - 回滚所需时间比较长
  - 执行时间长，容易造成主从延迟
- 处理大事务
  - 避免一次处理太多的数据
  - 移除不必要在事务中的 `SELECT` 操作



数据库备份远程同步计划任务

**教训：** 最好不要在主库上数据库备份； 大型活动取消这类计划





CPU

不支持多CPU对统一SQL的并发处理

CPU密集型

系统的并发量

64位架构CPU上运行32位的服务器版本



将多次写操作合并为一次





## 基础

### 函数

#### 单行函数

#### 分组函数

> 用于统计使用

sum 、 avg 、 max 、 min 、 count



排序 order by

### 分组查询

group by









## 事务











## 索引

> 帮助MySQL高效获取数据的 排好序的数据结构。

数据结构

- 二叉树
- 红黑树
- hash表
- b-tree

节点node 为key-value结构

二叉树特性：

- 右边的子元素 大于 父节点； 左边的子元素 小于 父节点。

可能出现 链表 形式



红黑树

二叉平衡树

数据量过大，红黑树的 深度过深，查找相当费性能



b-tree

使 一个节点存储更多的索引元素

> 叶节点 具有相同的深度，叶节点的指针为空
>
> 所有索引元素不重复
>
> 节点中的数据索引 从左到右递增排列



b+tree

多叉平衡树

> 非叶子节点 不存储data， 只存储索引(冗余)， 可以放更多的索引，没存储磁盘对应文件地址指针
>
> 叶子节点包含所有索引字段
>
> 叶子节点用指针连接，提高区间访问的性能
>
> 叶子节点之间的指针 是为了支持范围查找

把所有的索引元素在 叶子节点 存了一份完整的元素

把处于中间位置的索引元素 放到 非叶子节点 作为冗余来存放

存储的大小有限制，不存储 data，就可以 存储更多的 索引元素



hash表

不适于 范围查询，模糊查询



MyISAM 索引文件和数据文件 是分离的( 非聚集 )

非聚集： 索引和数据分开存储

InnoDB 索引文件和数据文件 是合并的( 聚集 )

聚集： 索引和数据聚集在同一个文件里面

> 表数据文件本身 就是按b+tree阻止的一个 索引结构文件
>
> 聚集索引 - 叶节点包含了 完整的数据记录

**InnoDB表 必须有主键，推荐使用整型的自增主键？**

整型

- 索引比较大小更快

- 占用空间更小

自增

- 不自增会造成 节点的分裂，树还会进行自动的平衡，会存在性能开销

**非主键索引结构 叶子节点存储的是主键值？**( 一致性和节省存储空间 )



联合索引的底层存储结构是什么样的

> 多个字段怎么进行排序
>
> 按字段先后 逐个取比对

索引最左前缀原则

要用索引 需要从第一个索引字段去用 才会去走联合索引

如果不是从第一个索引字段去 查询 ， 其他索引从 节点内部看是已经排好序的，但是从整个表内部看是没有 排好序的。



索引失效

最左前缀法则

使用范围查询时，范围查询的右侧欸点列索引会失效

查询字符串类型的字段不带单引号，则索引会失效

- db 中 数值类型会自动转换为字符串类型



数据库的精度问题

float和double形式是以 浮点数来进行存储

decimal 是以字符的形式来存储小数

高精度需要使用SQL Server数据库







## 主从架构

> 保证 各个数据库内的数据是一致的

原理

主从同步

主节点中 数据保存到 binlog 文件，存储了master节点中 所有的增删改的SQL语句

从节点中 存在两个线程

- I/O Thread
  - 从 主节点 拷贝数据过来， 放入 relay binlog 中继日志文件中
- SQL Thread
  - 从 relay binlog 读取，存储到 slave节点

缺点

数据的一致性 和 延迟



#### 常用架构

**一主一从**

- 不是 提高数据的性能，二是提高数据的可用性。(主挂掉从替换上)

**不能代替 数据备份**

- 主中DELETE， 从也会同步

**一主多从**

从节点 不能过多，会对 主节点产生 大压力。一般是 2-4 个

**双主**

双主数据 不重复

写性能 需要大

**级联同步**

主节点 压力过大

**环形多主**

#### 读写分离架构

**代理节点**

对请求进行分发，写master；读slave

##### 主从同步

master配置my.conf

- server-id 实例ID,不能和集群中的其他MySQL实例相同
- binlog
- binlog-to-db 要同步的数据库
- binlog-ignore-db

slave配置my.conf

- server-id
- [logbin] 还需要往其他节点进行同步时
- replicate-to-db 需要同步的数据库
- replicateignore-db

动态配置slave连接master

配置前必须给关闭slave服务

```
stop slave
```

主节点连接信息

```
change masterto master_host="master", master_user="root", master_password="123456";
```

启动slave服务

```
start slave
```

查看slave服务

```
show slave status
```

##### 基于docker编排MySQL集群

容器里面不允许有数据的存储

- 安全问题： 容器重新创建 会使数据丢失
- 性能问题： 容器里面文件驱动和操作系统文件驱动是不同的。

































b-tree索引

- 以 b+树的结构存储数据

特点

- 能够加快数据的查询速度
- 更适合进行范围查找( 顺序查找 )

使用情况

- 全值匹配查询
- 匹配最左前缀的查询
- 匹配列前缀查询
- 匹配范围值查询
- 精确匹配左前列并范围匹配另外一列
- 只访问索引的查询

使用限制

- 如果是不按照索引最左列开始查找，则无法使用索引
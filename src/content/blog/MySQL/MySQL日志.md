---
title: MySQL日志
publishDate: 2022-12-05
description: 'MySQL详解：MySQL Undo、Redo、Bin Log讲解'
tags:
  - 数据库
  - MySQL
  
language: '中文'
---

# MySQL Log

MySQL 作为一款成熟的关系型数据库管理系统，通过多种日志机制保障数据的持久性、一致性、可恢复性以及提供运维和性能分析支持。

本文将系统讲解 MySQL 中的核心日志类型，包括错误日志、**慢查询日志**、通用查询日志、**二进制日志（Binlog）**、中继日志（Relay Log）以及 **InnoDB 存储引擎特有的重做日志（Redo Log）和回滚日志（Undo Log）。**



## 错误日志（Error Log）

错误日志是 MySQL 最基础的日志，记录 MySQL 服务器启动、运行和停止过程中发生的严重错误、警告信息以及其他重要事件。

### 记录内容

- MySQL 服务启动和关闭信息
- 无法加载插件或配置错误
- 主从复制错误
- InnoDB 存储引擎的启动和崩溃恢复信息
- 严重的运行时错误（如磁盘空间不足、权限问题等）

### 配置与管理

- 默认启用，无法关闭。

- 日志文件路径由 `log_error` 系统变量指定。若未显式设置，通常位于数据目录下，文件名为 `hostname.err`。

- 可通过以下命令查看当前错误日志位置：

  ```sql
  SHOW VARIABLES LIKE 'log_error';
  ```

### 用途

- 排查数据库启动失败原因
- 监控数据库运行状态
- 定位严重错误和异常行为

---





## 通用查询日志（General Query Log）

通用查询日志记录所有发送到 MySQL 服务器的客户端请求，包括连接、断开连接以及执行的 SQL 语句（无论是否成功）。

### 记录内容

- 客户端连接和断开事件
- 所有执行的 SQL 语句（SELECT、INSERT、UPDATE、DDL 等）
- 语句执行时间戳

### 配置与管理

- 默认关闭（出于性能考虑）。

- 启用方式：

  ```sql
  SET GLOBAL general_log = 'ON';
  ```

- 日志输出目标由 `log_output` 变量控制，可写入文件（FILE）或表（TABLE，即 `mysql.general_log` 表）。

- 文件路径由 `general_log_file` 指定。

### 注意事项

- 对性能影响较大，尤其在高并发场景下，不建议在生产环境长期开启。
- 可用于审计或短期调试，但需谨慎使用。

---





## 慢查询日志（Slow Query Log）

慢查询日志用于**记录执行时间超过指定阈值的 SQL 语句**，是性能优化的重要工具。

### 记录内容

- 执行时间超过 `long_query_time` 秒的 SQL 语句
- 未使用索引的查询（若启用 `log_queries_not_using_indexes`）
- 查询执行计划相关信息（如扫描行数、返回行数等）

### 配置与管理

- 默认关闭。

- 关键参数：

  - `slow_query_log`：是否启用（ON/OFF）
  - `slow_query_log_file`：日志文件路径
  - `long_query_time`：慢查询阈值（单位：秒，默认 10 秒）
  - `log_queries_not_using_indexes`：是否记录未使用索引的查询

- 启用示例：

  ```sql
  SET GLOBAL slow_query_log = 'ON';
  SET GLOBAL long_query_time = 1; -- 记录执行超过1秒的查询
  ```

### 用途

- 识别性能瓶颈 SQL
- 优化索引设计
- 分析查询执行效率





## 中继日志（Relay Log）

中继日志仅在从库（Slave）上存在，用于临时存储从主库接收的二进制日志事件。

### 工作流程

1. 从库 I/O 线程连接主库，读取 binlog 事件。
2. 将事件写入本地 relay log。
3. 从库 SQL 线程读取 relay log 并重放，实现数据同步。

### 文件结构

- 文件名格式：`hostname-relay-bin.nnnnnn`
- 包含 `relay-log.info` 文件，记录当前应用到的位置。

### 用途

- 解耦主从复制的数据传输与应用过程，提高复制稳定性。
- 支持复制延迟监控和故障恢复。

-----



# 核心 Log

InnoDB 作为 MySQL 默认存储引擎，拥有自己独立的日志系统，**主要包括重做日志（Redo Log）和回滚日志（Undo Log）**，用于实现**事务的持久性和原子性**。

## 撤销日志（Undo Log）

Undo即撤销的意思，但咱们通常也习惯称它为回滚日志，在日常开发过程中，如果代码敲错了，一般会习惯性的按下Ctrl+Z撤销，而Undo-log的作用也是如此，但它是用来给MySQL撤销SQL操作的。

- **事务回滚 Rollback**：执行 ROLLBACK 时，利用 Undo Log 恢复数据到修改前状态，从而保证事务的**原子性**——即事务要么全部成功，要么全部失败。
- **多版本并发控制 MVCC 支持**：在可重复读（REPEATABLE READ）或读已提交（READ COMMITTED）隔离级别下，提供一致性读视图（快照读）。此时，系统通过 Undo Log 构建数据的历史版本，使当前事务能够看到符合其启动时间点的一致性数据视图，避免读写冲突，提升并发性能。



### Undo Log 存储文件

> 我们说一条写SQL执行前，会生成对应的反SQL记录在Undo-log中，但实际上并不会生成反SQL，这样去叙述仅是为了方便大家理解罢了

**那咱们怎么证明不会生成反SQL呢？**

+ 如果大家有仔细研究过MySQL的日志，应该会发现Undo-log并不存在单独的日志文件，也就是磁盘中并不会存在xx-undo.log这类的文件

**那Undo-log存在哪儿呢？**

+ InnoDB默认是将Undo-log存储在`xx.ibdata`共享表数据文件当中，默认采用段的形式存储。
+ 也就是当一个事务尝试写某行表数据时，首先会将旧数据拷贝到`xx.ibdata`文件中，将表中行数据的隐藏字段：`roll_ptr`回滚指针会指向`xx.ibdata`文件中的旧数据，然后再写表上的数据。

**那Undo-log究竟在xx.ibdata文件中怎么存储呢？**

+ 在共享表数据文件中，有一块区域名为Rollback Segment回滚段，每个回滚段中有1024个Undo-log Segment，每个Undo段可存储一条旧数据，而执行写SQL时，Undo-log就是写入到这些段中。
+ 不过在MySQL5.5版本前，默认只有一个Rollback Segment，而在MySQL5.5版本后，默认有128个回滚段，即支持128*1024条Undo记录同时存在。



### Undo Log 版本链

一条记录的每一次更新操作产生的 undo log 格式都有一个 roll_pointer 指针和一个 trx_id 事务id：

+ 通过 **trx_id** 可以知道该记录是被**哪个事务修改的**
+ 通过 **roll_pointer** 指针**可以将这些 undo log 串成一个链表**，这个链表就被称为版本链

![](https://cdn.nlark.com/yuque/0/2022/webp/12995622/1670229904575-6eb6b890-d73c-46d1-8522-4406452ed44c.webp)







### Undo Log 缓冲区

InnoDB在MySQL启动时，会在内存中构建一个`BufferPool`

**而这个缓冲池主要存放两类东西**

+ 一类是数据相关的缓冲，如索引、锁、表数据等
+ 另一类则是各种日志的缓冲，如Undo、Bin、Redo....等日志。

而当一条写SQL执行时，不会直接去往磁盘中的xx.ibdata文件写数据，而是会写在undo_log_buffer缓冲区中，因为工作线程直接去写磁盘太影响效率了，写进缓冲区后会由后台线程去刷写磁盘。

> 那么如果当一个事务提交时，Undo的旧记录会不会立马被删除呢？因为事务都提交了，不需要再回滚改动过的数据，似乎用不上Undo旧记录了，对吗？确实如此，但不会立马删除Undo记录，对于旧记录的删除工作，InnoDB中会有专门的purger线程负责，purger线程内部会维护一个ReadView，它会以此作为判断依据，来决定何时移除Undo记录。
>
> 
>
> 为什么不是事务提交后立马删除Undo记录呢？因为可能会有其他事务在通过快照，读Undo版本链中的旧数据，直接移除可能会导致其他事务读不到数据，因此删除的工作就交给了purger线程。





### Rollback原理

Undo Log 实现 Rollback（回滚）功能的核心机制，是通过**记录事务修改数据前的原始状态（旧值）**，并在需要回滚时，**逆向重放这些旧值以恢复数据到事务开始前的状态**。

当一个事务对数据进行修改（INSERT、UPDATE、DELETE）时，InnoDB 并不会立即覆盖原始数据，而是：

1. **先将原始数据（或足够恢复的信息）写入 Undo Log**；
2. **再在内存中（Buffer Pool）修改数据页**；
3. **事务提交前，这些修改仅存在于内存和 Redo Log 中，尚未持久化到磁盘数据文件**。

如果事务需要回滚（显式 ROLLBACK 或因异常终止），InnoDB 就根据 Undo Log 中保存的旧值，**反向执行操作**，将数据恢复原状。



#### 按操作类型分解回滚机制

##### INSERT

- **操作过程**：插入一行新记录。
- **Undo Log 内容**：记录该行的主键值（用于定位）。
- **回滚动作**：执行 **DELETE** 操作，将刚插入的行物理删除。
- **原因**：新插入的行在事务提交前对其他事务不可见，回滚只需清除该行。

> 示例：  
> 事务 T1 执行 `INSERT INTO t(id, name) VALUES (100, 'Alice')`；  
> Undo Log 记录：`主键=100`；  
> 回滚时：InnoDB 执行 `DELETE FROM t WHERE id = 100`。



##### DELETE

- **操作过程**：InnoDB 并非立即物理删除行，而是打上“删除标记”（delete mark）。
- **Undo Log 内容**：记录被删除行的完整旧值（包括所有列）。
- **回滚动作**：清除删除标记，并将行数据恢复为原始状态（即“取消删除”）。
- **本质**：将逻辑删除逆转为未删除状态。

> 示例：  
> 事务 T1 执行 `DELETE FROM t WHERE id = 100`；  
> Undo Log 记录：`(id=100, name='Alice')`；  
> 回滚时：InnoDB 将该行的 delete mark 清除，并恢复 name 字段。



##### UPDATE

- **操作过程**：修改某行的一个或多个字段。
- **Undo Log 内容**：记录被修改字段的**旧值**（以及主键用于定位）。
- **回滚动作**：将这些字段的值重新写回旧值。

> 示例：  
> 原始行：`(id=100, name='Alice', age=25)`；  
> 执行 `UPDATE t SET name='Bob', age=30 WHERE id=100`；  
> Undo Log 记录：`主键=100, name='Alice', age=25`；  
> 回滚时：InnoDB 执行 `UPDATE t SET name='Alice', age=25 WHERE id=100`。



#### 回滚的具体执行流程

结合上述纠正后的内容，咱们再对事务的回滚原理稍作更正，实际上当一个事务需要回滚时，本质上并不会以执行反SQL的模式还原数据，而是直接将roll_ptr回滚指针指向的Undo记录，从xx.ibdata共享表数据文件中拷贝到xx.ibd表数据文件，覆盖掉原本改动过的数据。

![](https://cdn.nlark.com/yuque/0/2022/webp/12995622/1670230029317-a0264f6f-2eae-4342-b704-8e1ad70d9a1d.webp)

当执行 `ROLLBACK` 或事务因崩溃需要回滚时，InnoDB 按以下步骤操作：

**步骤 1：定位事务的 Undo Log**

- 每个事务在开始时会被分配一个唯一的事务 ID（TRX_ID）。
- 事务的所有 Undo Log 记录都关联此 TRX_ID。
- InnoDB 通过事务控制块（trx_t 结构）找到其对应的 Undo Segment 和起始 Undo Page。

**步骤 2：从后往前遍历 Undo Log**

- Undo Log 按操作顺序写入，形成链表。
- 回滚时**逆序执行**（LIFO：后进先出），确保操作可逆。
  - 例如：先 UPDATE A，再 UPDATE B；回滚时先恢复 B，再恢复 A。

**步骤 3：根据 Undo 记录类型执行反向操作**

- 解析每条 Undo 记录的类型（INSERT/UPDATE/DELETE）；
- 调用对应的回滚函数：
  - `row_undo_ins()`：处理 INSERT 回滚（执行删除）
  - `row_undo_upd()`：处理 UPDATE 回滚（恢复旧值）
  - `row_undo_del()`：处理 DELETE 回滚（取消删除标记）

**步骤 4：更新数据页并写 Redo Log（保障崩溃安全）**

- 回滚操作本身也是“写操作”，因此：
  - 修改 Buffer Pool 中的数据页；
  - 生成对应的 **Redo Log**，确保即使在回滚过程中崩溃，重启后仍能继续完成回滚。
- 这体现了 InnoDB 的 **WAL（Write-Ahead Logging）** 原则：任何数据页修改前，必须先写 Redo Log。

**步骤 5：释放资源并标记事务结束**

- 回滚完成后，释放该事务占用的锁、内存结构；
- 标记事务状态为 `ROLLBACK`；
- Insert Undo Log 可立即回收（Update Undo Log 需等待 Purge）。



### MVCC 原理

通过 ReadView + undo log 实现 MVCC（多版本并发控制）

对于「读提交」和「可重复读」隔离级别的事务来说，它们的快照读（普通 select 语句）是通过 Read View + undo log 来实现的，它们的区别在于创建 Read View 的时机不同：

+ 「读提交」隔离级别是在每个 select 都会生成一个新的 Read View，也意味着，事务期间的多次读取同一条数据，前后两次读的数据可能会出现不一致，因为可能这期间另外一个事务修改了该记录，并提交了事务。
+ 「可重复读」隔离级别是启动事务时生成一个 Read View，然后整个事务期间都在用这个 Read View，这样就保证了在事务期间读到的数据都是事务启动前的记录。

![](https://cdn.nlark.com/yuque/0/2022/webp/12995622/1670230400117-07d33f47-3d44-48c3-94aa-233c058fc3e1.webp)

**MVCC具体详细解释，可以看 MySQL 事务篇章**

--------





## 重做日志（Redo Log）

之前MySQL每一条更新记录，都需要写入磁盘，然后磁盘也要找到对应的那条记录，然后再更新，整个过程 IO 成本、查找成本都很高。

为了解决这个问题，MySQL 的设计者就用了技术解决该问题。（指的是 MySQL 的写操作并不是立刻更新到磁盘上，而是先记录在日志上，然后在合适的时间再更新到磁盘上）







![](https://cdn.nlark.com/yuque/0/2022/png/12995622/1670230852270-bbf37c9d-2a75-4ecf-9b16-0130a1c5d51d.png)

### 介绍

Redo Log（重做日志）是 InnoDB 存储引擎实现事务**持久性（Durability）** 和**崩溃恢复（Crash Recovery）** 的核心机制。它采用 **WAL（Write-Ahead Logging，预写日志）** 技术，确保即使数据库因意外宕机而中断，已提交事务的修改也不会丢失。

其实就是一块固定大小的重做物理日志文件，可以循环写。redo log记录的是“在某个数据页上做了什么修改

### Write Ahead Logging

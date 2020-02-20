# zookeeper与Paxos
并没有直接使用Paxos，而是ZAB协议，zookeeper atomic Broadcast。

## 什么是zookeeper
zookeeper典型的分布式解决方案。
* 顺序一致性
* 原子性
* 单一视图：无论客户端连接哪一个服务器，看到的数据模型都是一致的。
* 可靠性
* 实时性：仅仅保证一段时间内，最终读到最新的数据状态。
  
## 基本概念
### 集群角色
* Leader 读写服务
* Follower 读服务，参与选举
* Observer 读服务，不参与选举
### 会话
Session简单说就是一个tcp连接，在超时时间连接上，还是属于一个会话。sessionTimeout设置
### Znode
节点分为两类：
* 第一类为构成集群的机器，机器节点
* 第二类为数据模型的数据单元，数据节点，分为临时节点，和持久节点。

### 版本
ZNode会维护Stat数据结构，三个版本：version当前版本，cversion子节点版本，aversion当前ACL版本

### Watcher
事件监听器

### ACL
* CREATE 创建**子节点**权限
* READ
* WRITE
* DELETE 删除**子节点**权限
* ADMIN 设置节点acl的权限

## ZAB协议
一种主备模式的系统架构。一个主进程来广播状态变更。

所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而余下的其他服务器则成为Follower服务器。Leader服务器负责将一个客户端事务请求转换成一个事务Proposal（提议），并将该Proposal分发给集群中所有的Follower服务器。之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确的反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交。

ZAB包括两种基本模式：崩溃恢复，消息广播。

一个由3台机器组成的集群，一个Leader，2个Follower，其中一个Follower挂了，依然可以运行，Leader自己也能获得过半支持，因为Leader支持自己。

### 消息广播
消息广播类似二阶段提交模式，ZAB协议移除了二阶段提交过程中的中断逻辑，意味着过半的Ack后，就开始提交Proposal。这种简化的模型无法应对Leader崩溃，所以有了崩溃恢复模式。整个消息是基于有FIFO特性的TCP协议进行通信的。顺序性得到保证。

广播前会为Proposal分配一个全局唯一id，及事务ID（ZXID），按zxid先后顺序处理。

### 崩溃恢复
恢复后选举出一个Leader，整个集群都可以感知到这个新Leader。


### 基本特性
ZAB的一些特性：

1、需要保证那些已提交的事务最终被所有的服务器提交。 比如发出commit前，Leader崩溃了。

2、确保丢弃那些只在Leader服务器上被提出的事务。server1作为Leader提了一个Proposal-1就崩溃了,其他机器没有收到这个Proposal-1，当server1从新加入到集群中确保丢弃Proposal-1。

最好选举出拥有最高编号事务的机器成为Leader，可以省去检查Proposal的提交和丢弃工作。

### 数据同步
事务编号zxid，设计为64位数字，低32位是简单的单调递增操作，高32位代表了Leader周期epoch的编号。

## 深入ZAB
细分可以分为三个阶段，发现，同步，广播，循环执行这三个阶段，一个循环为一个主进程周期。

三种状态：
* looking leader选举阶段
* followering follower和leader保持同步状态
* leading leader作为主进程领导状态


## ZAB与Paxos的联系与区别
zab并不是paxos的一个典型实例。本质区别，设计目标不一样，zab构建高可用的分布式数据主备系统。paxos构建一个分布式的一致性状态机系统。





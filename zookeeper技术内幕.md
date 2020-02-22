# 系统模型
数据模型，节点特性，版本，Watcher，ACL

## 数据模型
ZNode,形成的结构化的命名空间，称之为树。

![tree](images/tree.png)  

事务id：在zookeeper事务指的是可以改变状态的操作。会分配事务id——ZXID，64位。

## 节点特性
### 节点类型
节点分为：持久节点——persistent，临时节点——ephemeral，顺序节点——sequential。可以组合：

* 持久节点
* 持久顺序节点
* 临时节点——临时节点只能作为叶子节点
* 临时顺序节点

### 状态信息
get命令可以获取状态信息，保存在Stat。
![stat](images/stat.png)

## 版本
zookeeper中的版本和传统意义上的版本有区别。它表示的是修改次数。  
乐观锁可以用version做写入校验

## Watcher
主要包含客户端线程，客户端WatcherManager和zookeeper服务器三部分。  
wather通知状态与事件一览  
![wather](images/watcher.png)

注意：
1、NodeDataChanged，即使数据变更内容一样，也会触发这个事件。  
2、NodeChildrenChanged，子节点数量和组成变更。子节点内容变更不会触发。
3、AuthFailed，指的是授权失败。

### 回调方法  
只是单纯的通知，不传数据，需要自己再去获取。
### 工作机制
客户端注册Watcher，服务端处理Wather和客户端回调watcher

## ACL
和linux的acl有一些区别，可以从三个方面理解：权限模式——Scheme，授权对象——id和权限——Permission。通常使用“scheme：id：permission”来标识一个有效的ACl信息。
### 权限模式
用来确定权限验证过程中使用的检验策略。使用最多的四种：
* IP
* Digest——类似于username：password
* world——最开放的权限控制，一种特殊的Digest：“world：anynoe”
* Super——超级用户

### 授权对象：ID
指一个实体或用户，模式不同id类型不同。比如——IP：192.168.1.1
### 权限
五大类：create，delete，read，write，admin

### 权限扩展
插件

### super模式
创建了持久节点，客户端退出了，也不用这个节点了，且有acl，可以用super模式删除。


# 序列化与协议
Jute序列化组件。

# 客户端
几个核心组件
* Zookeeper实例：客户端入口
* ClientWatchManager：客户端Watcher管理器
* HostProvider：客户端地址列表管理器
* ClientCnxn：客户端核心线程，内部又包含两个线程，SendThread和EventThread。

启动大体分为3个步骤：  
1、设置默认的Wather  
2、设置Zookeeper服务器地址列表  
3、创建ClientCnxn  


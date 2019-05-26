# zookeeper

## 什么是Zookeeper

​	Zookeeper是一种用于分布式应用程序的高性能**协调**服务，提供一种集中式信息存储服务

### 特点

​	数据存储在内存中，类似文件系统的树形结构，高吞吐量和低延迟，集群高可靠

### 作用

​	基于Zookeeper可以实现分布式统一配置中心，服务注册中心，分布式锁等功能

### 应用案例

​	Hbase：使用Zookeeper进行Master选举，服务间协调

​	Solr：使用Zookeeper进行集群管理，Leader选举，配置管理

​	dubbo：服务注册

​	Mycat：集群管理，配置管理

​	Sharding-sphere：集群管理，配置管理

### 同类产品

​	consul，etcd(轻量级zookeeper)，doozer

## 什么是分布式协调服务

​	单机系统因处理能力上限，可用性，可靠性的考虑，才扩展成分布式系统

​	原来在单机进程中完成的一件事的多个步骤，变成在多个计算机中完成。这时就需要协调各个节点做事的顺序。原来在单系统中资源竞争通过锁进行同步控制；现在变成了进程间的资源竞争，所以需要分布式协调。

​	我们可以把每个分布式系统中需要的协调管理的公共基础部分抽取出来作为一个基础公共服务供大家使用。这就是分布式协调服务

## Zookeeper的安装和使用

​	安装网上教程很多，略

​	**命令**

| 指令         | 描述                                |
| ------------ | ----------------------------------- |
| ls/ls2       | 获取子节点/查看子节点详细信息       |
| create       | 在zookeeper中的某个位置创建一个节点 |
| exists       | 测试节点是否存在                    |
| delete       | 删除节点                            |
| get data     | 从指定节点读取数据                  |
| set data     | 将数据存入指定节点                  |
| get children | 查询指定节点之下的所有子节点        |
| sync         | 等待数据进行同步                    |

​	**Java API**

​	核心类：Zookeeper

| 方法         | 描述                        |
| ------------ | --------------------------- |
| connect      | 连接到Zookeeper集合         |
| create       | 创建znode                   |
| exists       | 检查znode是否存在及其信息   |
| get data     | 从特定的znode获取数据       |
| set data     | 在特定的znode中设置数据     |
| get children | 获取特定znode中的所有子节点 |
| delete       | 删除特定的znode及其所有子项 |
| close        | 关闭连接                    |

## Zookeeper核心概念

![](http://prvyof0n9.bkt.clouddn.com/10.png)

### Session会话

- 一个客户端连接一个会话，由zk分配唯一会话id。
- 客户端以特定的时间间隔发送心跳以保持会话有效(tickTime)。
- 超过会话超时时间未收到客户端的心跳，则判断客户端死了(默认两倍tickTime)。
- 会话中的请求按FIFO顺序执行(First-In-First-Out)。

## 数据模型

### 层次名称空间

- 类似unix文件系统，以/为根
- 区别：节点可以包含与之关联的数据以及子节点(即是文件也是文件夹)
- 节点的路径总是表示为范围的、绝对的、斜杠分隔的路径

### znode

- 名称唯一，命名规范
- 节点类型：持久、顺序、临时、临时顺序
- 节点数据构成

![](http://prvyof0n9.bkt.clouddn.com/11.png)

### znode-节点类型

| 节点类型     | 创建命令                                                     |
| ------------ | ------------------------------------------------------------ |
| 持久节点     | create /app1 666                                             |
| 临时节点     | create -e /app2 888                                          |
| 顺序节点     | create -s /app1/cp 888 名称为:cp0000000000<br />create -s /app1/aa 名称为0000000001 |
| 临时顺序节点 | create -e -s /app/888                                        |

顺序节点中：

- 只在一次会话中生效
- 创建出的节点是10位10进制序号
- 每个父节点有一个计数器
- 计数器是带符号的int(4字节)到2147483647之后将溢出(导致名称<path> -2147483647)

### znode-数据构成

- 节点数据：存储的协调数据(状态信息、配置、位置信息等)
- 节点元数据(stat结构)
- 数据量上限：1M

### znode-元数据stat结构

| Stat结构字段   | 描述                                            |
| -------------- | ----------------------------------------------- |
| cZxid          | 创建该节点zxid                                  |
| mZxid          | 最后修改该节点的zxid                            |
| pZxid          | znode最后更新的子节点zxid                       |
| ctime          | 该节点的创建时间                                |
| mtime          | 该节点的最后修改时间                            |
| dataVersion    | 该节点数据被修改的次数                          |
| cversion       | 该节点的子节点变更次数                          |
| ephemeralOwner | 临时节点的所有者会话id，如果不是临时节点，则为0 |
| dataLength     | 该节点的数据长度                                |
| numChildren    | 子节点数                                        |

### ZooKeeper中的时间

#### 多种方式跟踪时间

- Zxid：zookeeper中的每次更改操作都对应一个唯一的事务id，成为Zxid，它是一个**全局有序**的戳记，如果zxid1小于zxid2，则zxid1发生在zxid2之前。
- Version numbers：版本号，对节点的每次更改都会导致该节点的版本号之一增加。
- Ticks：当使用多服务器zookeeper时，服务器使用“滴答”来定义事件的时间，如上传状态、会话超时、对等点之间的连接超时等。滴答时间近通过最小会话超时(滴答时间的2倍)间接公开；如果客户端请求的会话超时小于最小会话超时，服务器将告诉客户端会话超时实际上是最小会话超时。
- Real time：zookeeper除了在znode创建和修改时将时间戳放入stat结构之外，根本不使用Real time或时钟时间。

## Watch监听机制

客户端可以在znodes上设置watch，监听znode变化

如何设置监听watch

- stat path [watch]例如：stat /app1 1
- ls path [watch]
- ls2 path [watch]
- get path [watch]

JavaAPI实现监听

- getData()
- getChildren()
- exists()

### 两类Watch

- data watch 监听数据变化：get
- child watch 子节点变化 ls, ls2

### 触发Watch事件

- Created event：exists()
- Deleted event：exists(), getData(), getChildren()
- Changed event：exists(), getData()
- Child event：getChildren()

### Watch重要特性

- 一次性触发：watch触发后即被删除，要持续监控变化，则需要持续设置watch。
- 有序性：客户端先得到watch通知，后才会看到变化结果。

### Watch注意事项

- watch是一次性触发器；如果获得了一个watch事件并希望得到关于未来更改的通知，则必须设置另一个watch。
- 因为watch是一次性触发器，并且在获取事件和发送获取watch的新请求之间存在延迟，所以不能可靠地得到节点发生的每个改变。
- 一个watch对象只会被特定的通知触发一次。如果一个watch对象同时注册了exists、getData，当节点被删除时，删除事件对exists、getData都有效，但只会调用watch一次。

## ZooKeeper特性

- 顺序一致性：保证客户端操作是按顺序执行的。
- 原子性：更新成功或失败，没有部分结果。
- 单个系统映像：无论连接到哪个服务器，客户端都将看到相同的内容。
- 可靠性：数据的变更不会丢失，除非被客户端覆盖修改。
- 及时性：保证系统的客户端当时读取到的数据是最新的。

## ZooKeeper应用场景

### 数据发布订阅(配置中心)

#### 用zookeeper实现配置中心

- znode能存储数据
- Watch能监听数据改变



### 命名服务

### Master选举

### 集群管理

### 分布式队列

### 分布式锁

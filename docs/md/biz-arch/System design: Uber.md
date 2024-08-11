# System design: Uber





## 1 简介

**Uber** 是一款为用户提供**叫车服务**的应用程序。 任何需要搭车的人都可以注册并预订车辆从源地到目的地旅行。 拥有车辆的任何人都可以注册成为司机,并将乘客送到目的地。 司机和乘客可以通过他们的智能手机上的 Uber 应用程序进行交流。

## 2 需求

### 2.1 功能性

- **更新司机位置**:司机是一个移动实体,因此应该以固定的间隔自动更新司机的位置。
- **查找附近的司机**:系统应该查找和显示附近可用的司机给乘客。  
- **请求行程**:乘客应该能够请求行程,之后最近的司机应该会收到乘客的请求通知。
- **管理支付**:在行程开始时,系统必须启动支付流程并管理支付。
- **显示司机预计到达时间(ETA)**:乘客应该能够看到司机的预计到达时间。
- **确认接送**:司机应该能够确认他们已经接送了乘客。  
- **显示行程更新**:一旦司机和乘客接受了乘车,他们应该能够持续看到行程更新,如 ETA 和当前位置,直到行程结束。
- **结束行程**:在到达目的地时,司机标记旅程完成,然后他们便可供下一程行程。

### 非功能性

- **可用性**:系统应高度可用。哪怕是一小部分秒的宕机也可能导致行程失败,导致司机无法定位乘客,或者乘客无法联系司机。
- **可扩展性**:随着时间的推移,系统应该能够扩展以处理越来越多的司机和乘客。  
- **可靠性**:系统应提供快速无误的服务。叫车请求和位置更新应该顺利进行。
- **一致性**:系统必须**强一致**。一个区域内的司机和乘客应该对系统有一致的视图。
- **欺诈检测**:系统应该能够检测与支付相关的任何欺诈活动。

## 3 设计



![img](https://miro.medium.com/v2/resize:fit:1100/1*dlv51Q5TO06kbUXk3iODjg.png)  

## 4 组件

### **位置管理器**

当用户打开应用程序时,该**服务向乘客显示附近的司机**。 此服务还**每四秒钟从司机那里接收位置更新**。 然后将司机的位置传达给四叉树服务,以确定司机在地图上的所属部分。 **位置管理器在数据库中保存所有司机的最后位置**,并在行程中保存司机所遵循的路线。

### 四叉树服务

**四叉树地图服务**更新司机的位置。 主要问题是如何有效地处理寻找附近司机。

四叉树有助于将地图划分为部分。 如果司机数量超过某个限制,例如 500,那么我们将该部分分割成四个更多的子节点,并将司机划分到其中。

四叉树中的每个叶节点都包含无法进一步划分的部分。 我们可以使用相同的四叉树来查找司机。 我们现在最大的区别在于,我们的四叉树不是为定期升级而设计的。 因此,我们的动态部分解决方案存在以下问题。

每当司机的位置更改时,我们都必须更新数据结构以指出所有活动司机都会每四秒钟更新一次位置。**每当一个司机的位置改变时,修改四叉树都需要很长时间。** 为了确定司机的新位置,我们必须首先根据司机以前的位置找到一个合适的格网。 如果新位置与当前网格不匹配,我们应该从当前网格中删除司机并将其移至正确的网格。

![img](https://miro.medium.com/v2/resize:fit:1100/1*Sh9Yev1-kWaZIW5fjzTxVQ.png)  

图 2.0:更新新的当前位置  

为了解决上述问题,我们可以使用哈希表来存储司机的最新位置,并偶尔更新我们的四叉树,比如 10-15 秒。 我们可以每 15 秒左右而不是每 4 秒更新四叉树中的司机位置,我们使用一个每 4 秒更新一次并反映司机最新位置的哈希表。 通过这样做,我们可以使用更少的资源和时间。

![img](https://miro.medium.com/v2/resize:fit:1100/1*-hMvmAZkWSOIszknXEpi8Q.png)  

图 3.0:哈希表每 4 秒更新一次,四叉树每 15 秒更新一次  

100 万司机需要每四秒发送一次更新的位置。 100 万次请求四叉树服务会影响其性能。 为此,我们首先将更新的位置存储在存储在 Redis 中的哈希表中。 最终,这些值每 10-15 秒复制到持久存储中。

### 请求行程

乘客联系**请求行程**服务以请求行程。 乘客在此添加目的地位置。 然后,请求车辆服务与查找司机服务联系,通过位置管理器服务预订车辆并获取车辆详细信息。

### 查找司机

**查找司机**服务查找可以完成行程的司机。它将所选司机的信息和行程信息发送回请求车辆服务,以向乘客传达详细信息。查找驱动程序服务还与行程管理器联系以管理行程信息。

### 行程管理器

**行程管理器**服务管理所有与行程相关的任务。 它在数据库中创建行程,并将行程的所有信息存储在数据库中。

### **ETA 服务**

**ETA 服务**处理预计到达时间。 当他们的行程被安排时,它向乘客显示接人 ETA。 该服务考虑路线和交通等因素。 给定公路网络上原点和目标,预测 ETA 的两个基本组件是:

- 计算原点到目的地的最短路线。  
- 计算行驶该路线所需的时间。  

要确定源和目的地之间的最短路径,我们可以利用诸如 Dijkstra 算法之类的路由算法。 然而,Dijkstra 或任何在未处理的图形之上运行的其他算法都对于这样的系统来说比较慢。

因此,在叫车平台运营的规模上,这种方法是不切实际的。  

为了解决这些问题,我们可以将整个图形划分为分区。 我们**使用收缩层次预处理分区内部的最佳路径**,并仅处理分区边界。 这种策略可以显着减少时间复杂度,因为它将图形划分为大体独立的细小单元层。必要时,将并行在分区中执行预处理阶段以增加速度。  

![img](https://miro.medium.com/v2/resize:fit:770/1*CZXVaiTEGzDJpZsOl3XJlQ.png)

图 4.0:对图进行分区  

### 数据库

根据我们的理解和要求(高可用性、高可扩展性和容错能力),我们可以使用 **Cassandra** 来存储**司机的最后已知位置和行程信息,行程结束后**,不会有**更新**。 我们使用 Cassandra,因为我们存储的数据量巨大,且不断增长。  

我们可以使用 **MySQL** 数据库来存储**正在进行的行程信息**。 我们对**正在进行的行程使用 MySQL 进行频繁更新**,因为行程信息是关系型的,并且需要在表之间保持一致。

### 模式

![](https://miro.medium.com/v2/resize:fit:770/1*gkypBWXih7nReYc-XkIO7A.png)  

图 5.0:存储模式  

## 5 评估

### 可用性

我们的系统高度可用。 我们使用了 WebSocket 服务器。 如果用户断开连接,会话将通过负载均衡器与不同的服务器重新创建。 我们使用了多份数据库副本,具有主从复制模型。 我们拥有 Cassandra 数据库,它提供高度可用的服务和没有单点故障。 我们使用了 CDN、缓存和负载均衡器,这提高了我们系统的可用性。

### 可扩展性

我们的系统具有很高的可扩展性。 我们使用了许多独立的服务,以便我们可以根据需要独立地水平扩展这些服务。 我们使用四叉树通过将地图划分为较小的部分来搜索,这缩短了我们的搜索空间。 我们使用 CDN,这增强了处理更多用户的能力。 我们还使用了 Cassandra NoSQL 数据库,它支持水平扩展。 此外,我们使用负载均衡器来通过在不同服务器之间分配读取工作负载来提高速度。  

### 可靠性

我们的系统高度可靠。 即使乘客或司机的连接中断,行程也可以继续。 这是通过使用他们的手机作为本地存储来实现的。 使用多个 WebSocket 服务器可以确保顺利、近乎实时的操作。 如果任何服务器失败,用户能够与另一个服务器重新连接。 我们还使用服务器和数据库的冗余副本来确保没有单点故障。 我们的服务是解耦和隔离的,这最终增加了可靠性。 负载平衡器帮助将请求从任何失败的服务器转移到正常的服务器。  

### 一致性

我们使用 MySQL 等存储来保持数据的全局一致性。 此外,我们的系统执行同步复制以实现强一致性。 由于行程的数据写入者和查看者有限(乘客、司机、一些内部服务),使用传统数据库不会成为瓶颈。 此外,在这种情况下,数据分片也更容易。
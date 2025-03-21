# 如何避免半夜崩溃？支付系统高可用性实践！

## 1 系统高可用性保障

某支付渠道出现异常的情况时，降级和熔断是我们常用的一种方式，但其对用户的还款体验会有较大的影响，且无法更精准的缩小其影响范围。因此根据支付渠道降级的业务策略来调控某一渠道异常后的路由权重，通过滑动窗口算法来统计和监控渠道某个时间段内异常的比例和绝对数量，达到一定阈值后负反馈给支付路由引擎，在其对支付渠道进行筛选和排序时降低异常渠道的权重，进而调节该异常渠道的使用情况。在降低影响的同时又不会因部分的异常全部关停该渠道的请求，达到智能调控。

### 1.1 精准高效的支付路由

为达到选出最优支付渠道及提高性能的效果，支付系统还设计一套支持多场景下的渠道选取的路由算法。

渠道决策树算法以经典决策树算法为核心思想，结合以路由策略形成的一种渠道路由排序算法。根据支付方式不同，收集在当前场景下要进行计算的条件作为节点，构建出一个树模型。然后将备选渠道从根节点进入，经由节点的筛选结果流向不同的分支，最终完成备选渠道的排序过程。

采用渠道决策树算法对渠道进行排序的优势：

- 节点生成灵活简单，可根据支付方式不同定制组合不同的排序条件拼装成一个决策树，这种拼装方式可扩展，在我们丰富路由策略时，可零代码快捷适配新的策略配置
- 高效，在决策排序过程中，利用其分类和归纳的特点在 O（nlogn）的复杂度下快速获得结果

![](https://codeselect.oss-cn-shanghai.aliyuncs.com/c3c31dbe8fbf6b995efe87955ab41e42.png)

渠道路由的策略包含人工配置的一些渠道客观条件，如业务支持银行信息、支付限额等，也有根据渠道的表现自动做出的权重策略调整，如渠道异常后的自动负反馈调节策略。

### 1.2 负反馈调节算法

监控渠道的健康情况，针对渠道异常时，统计异常情况并形成一个指标负反馈给支付路由，达到动态调整渠道权重。思想来自 Sentinel  一种限流算法滑动窗口算法，将时间窗口划分若干时间片段，每过一个时间片段时间窗口会向右滑动一格，每个时间片段都有独立计数器。所以统计整个时间窗口的请求数时只需累加所有的时间片段的数据。时间窗口划分越细，滑动越平滑，统计越精确。

#### 滑动窗口优点

- 无临界值问题
- 统计精度高
- 保证统计区间的连续性
- 渠道异常时可快速调节
- ...

![](https://my-img.javaedge.com.cn/javaedge-blog/2024/08/040369e2f5cc8c7cc7f450b7463ba685.png)

如负反馈调节算法以 10s 作为统计的时间窗口大小 LeapArray，并配置 5 个样本数 SimpleCount。即对应算法的时间窗口大小为 10s，包含 5 个时间片段，每个时间片段2s，存储的数据记作为指标桶 MetricBucket，其结构为支付渠道+结算主体+支付方式维度的响应数量及响应异常数量。其应用效果要分两阶段：

- 在渠道路由请求时，要根据当前时间按照窗口时长进行分段，拿到当前时间对应的分段 ID，并取出对应的窗口返回去给对应的统计逻辑进行阈值判断
- 需在渠道请求的拦截方法中，在得到请求响应后，使用同样计算方式得到当前时间窗口及时间片段，并通过业务条件判断对 total 和 error 等统计指标进行调整，将数据存入当前时间片段的指标桶

![](https://codeselect.oss-cn-shanghai.aliyuncs.com/cdcda03155cc16605fc2bd9f1a9231bd.png)

在渠道路由阶段得到窗口中的统计数据后需要和设定阈值条件作对比，得出当前渠道的健康权重分数。从两个维度提供阈值条件：

- 绝对数量，同一时间窗口内若有超过 10 条（该值可配置调整）异常的支付请求，则将异常请求的数量 n 除以该时间窗口 10 秒内总订单数量 m 得到结果 y1，乘以其权重基准值 10，得到结果 10y1
- 异常比例，即在同一时间窗口内若有超过 10%（该值可通过配置调整）的异常支付请求，则以同样的方式计算出异常订单比例结果为 y2，乘以其权重基准值 10，得到结果 10y2

最后通过公式 10-（10n/m）得到最终的系统权值评分，分值越高代表支付渠道的权重越高，排序越靠前。

## 2 系统自适应压力调节能力

任何系统都有业务处理的瓶颈上限，肉没法精准的压力识别，导致任务并发量或资源占用明显超过服务器支撑范围，则直接影响线上服务稳定。系统实现过程中，搭建压力检测服务，通过该服务动态识别机器、组件的运行压力，进行反向调节业务请求速率、并发等限制，以此来达到保护生产服务稳定性的目的。

![](https://codeselect.oss-cn-shanghai.aliyuncs.com/6f45d9db694f464e306d5b8d5f163a37.png)

压力检测技术，支付系统除自身服务节点，还关联多个组件服务。如 MySQL、RabbitMQ 。如何让系统能自动感知整个支付流程中某节点出现性能瓶颈呢？

压力指数是压力检测服务定义的一个性能指标，包含：

- 系统服务节点的压力指标
- 系统依赖的组件的一些核心指标，如服务节点的 JVM 内存、CPU 消耗，MySQL 的连接数、主从同步延迟，RabbitMQ 队列积压量

压力检测服务通过将以上多个维度的指标通过一定的权重比计算出压力指数，再根据全链路压测结果以及故障演练等方式总结出一个适当的阈值梯度。

压力检测服务计算出的压力指数达到一定的阈值梯度后，会通知支付系统进行限流处理。同时将该指数上报给系统调用方，达到从源头降低压力的效果。当压力指数一直处于高位并保持一段时间后，会触发系统告警，由人工介入排查压力情况是否正常。

## 3 组件降级提升系统高可用性

支付系统采用 RabbitMQ 作为异步消息组件，连通整个支付流程，选用RabbitMQ原因：

- 保障金融系的稳定性，需要防止消息丢失，要支持消息持久化
- 考虑业务的复杂性，需要支持队列的延迟特性
- 结合性能和稳定性的综合考量，以及组件的成熟度

Q：RabbitMQ在支付系统起总线作用，那就要考虑异常时的降级措施。

A：MQ不存储任何数据、无业务逻辑和复杂性的组件，集成在服务中的主要功能是解耦和削峰。替代方案中也需支持解耦和削峰功能的组件，同时尽量避免增加系统复杂度。筛选后，选取 Redis 作为降级替代组件，MQ异常时，舍弃部分 ack 机制，用 Redis 的 list 结构作为异步队列进行异步生产和消费，zset 结构作为延迟队列。支付系统在请求 RabbitMQ 发送消息多次重试都失败后，会将消息体存入 Redis 数组中，继续由 Redis 作为替代队列完成消息的生产与消费。同时，为了保证数据的完整性和一致性，我们还需要做到以下几点来完善 Redis 在整个流程中的 MQ 效果：

- MQ 异常后的快速感知和切换
- 动态调节 Redis 生产和消费的速率，防止 Redis 内存占用过大
- 做好消息的幂等消费

目前，支付系统大部分核心队列及全部非核心流程队列已支持 MQ 的故障切换，异常感知和切换时效达到3s内，为服务高可用性提供保障。

## 4 规划

未来流量激增，可横向扩充节点及自动压力调节和主动降级等方式，将支付处理能力快速提高 5-10 倍水平。
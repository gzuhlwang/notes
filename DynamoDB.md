论文：Amazon DynamoDB：A Scalable, Predictably Performant, and Fully Managed NoSQL Database Service

会议：USENIX ATC '22

# 6个系统性质

* 完全托管的NoSQL云数据库服务；
* 多租户架构；
* 无限的表规模；
* 可预测的性能（延迟）；
* 高可用。
普通表（单个区域跨多个可用区（AZ）复制）的可用度是4个9；全球表（跨多个AWS区域复制）的可用度是5个9。
* 灵活的使用案例。

# 1个共同主题
持久性、可用性、扩展性和可预测的性能。

# 4个经验
* 用户体验：适应客户流量模式，改组数据库表的物理分区方案。
* 高持久性方面：针对静态数据（data@rest）的持续验证是保护数据免遭硬件故障和软件Bug的最可靠方式。
* 高可用性方面：复杂算法（如，复制协议）的形式化证明，game days（混沌、压力、断电测试），组件级的升级/降级测试和部署安全性。
* 稳定性：相对于绝对效率，设计可预测的系统提升系统稳定性。

# DynamoDB架构

* 微服务架构。数十个微服务。
  * 核心服务有：元数据服务、请求路由服务（认证、授权、路由等）、存储服务、autoadmin服务（资源）、时间点恢复服务、按需备份。
  * 自动管理服务是中枢。依赖的其他服务：IAM、KMS。

* 一致性算法：Multi-Paxos，复制组的副本数是3，满足2f+1的要求。
* 领导租约机制
* 可用性 over 一致性。


一致性读（consistent read）：leader节点提供一致性读。
最终一致性读（eventually consistent reads）：复制组中的任意replica均可提供读。

# 挑战
## 部署
* 软件部署原因：新的feature，bugfix，性能优化等。
* 分布式部署挑战：新的软件可能会引入新的消息类型或协议变动使得系统中的老的软件不理解。
* 部署流程：读-写部署。首先部署能够读取新的的消息格式或协议，接着部署发送新消息的软件。打开新消息开关。
* 部署策略：灰度发布。设置可用度指标报警阈值。

## 可用性
原因1：没有健康的leader或quorum不足。

应对：添加日志节点（Log Replica），秒级；

原因2：leader故障检测误报

应对：最小化误报次数，DynamoDB中会向复制组中的节点发消息，询问其与leader节点的连通性。
若返回健康的leader消息，则不触发领导人选举。选举领导人以及新当选的领导人会等待原先的领导人租约（lease）到期。选举
领导人会中断可用性，新当选的领导存在一个无法对外提供读写服务的窗口期，通常为数秒。故障检测必需快速且有效。

# takeaways

* 可以在区块链引擎中应用形式化方法（如，验证共识协议的正确性）。
 * 可用的工具：TLA+、[apalache](https://github.com/informalsystems/apalache)等。
* 除了常规的单元测试、集成测试、性能测试，需要引入混沌测试来建立信心。
* 区块链服务需要考虑备份和恢复方案。复制 ≠ 备份。
* 区块链共识组件在检测leader故障或不响应时，应谨慎触发选举流程。
* 区块链服务（Blockchain as a service）应借鉴多租户架构，为客户节省成本，同时提高资源利用率。

# 小知识
* gray network failure：灰色网络故障。
* 备份和恢复：应对逻辑损坏。
* 备份：全备份。
* 恢复：时间点恢复（point-in-time recovery，PITR）。
* 中断（disruptions）。
* 误报（false positives，假阳性）。
* 物理介质损坏。

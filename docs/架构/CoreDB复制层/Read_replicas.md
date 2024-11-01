将数据异步复制到一个或多个只读副本集群

除了基于共识的核心分布式复制之外，CoreDB 还扩展了 Raft，添加了只读副本（也称为观察者节点），这些副本不参与写入，但以异步方式获取时间线一致的数据副本。

只读副本是 Universe 中主数据的只读扩展。 通过只读副本，Universe 的主要数据可以跨一个区域（region）中的多个可用区（zone）或附近的区域（region）进行复制。 读取副本不会增加写入延迟，因为写入不会同步将数据复制到它们。 相反，数据会异步复制到只读副本。

远程数据中心的节点可以通过只读方式添加。 这通常用于某些工作负载无法容忍基于分布式共识写入的延迟的情况。

**1.复制因子**
每个 AiSQL Universe 都包含一个主数据集群和一个或多个只读副本集群。 因此，每个只读副本集群可以独立地拥有自己的复制因子。

只读副本集群的复制因子也可以是偶数。 例如，复制因子为 2 的只读副本集群是完全有效的。 这是因为只读副本不参与 Raft 共识操作，因此不需要奇数个副本来保证正确性。

**2.写入只读副本**
应用程序可以向只读副本发送写入请求，但这些写入请求在内部重定向到主机群对应的Tile leader。 这是可能的，因为只读副本了解 Universe 的拓扑。

**3.架构变更**
由于只读副本是 Raft 复制级别的扩展，因此架构更改会透明地应用于这些副本。 无需在只读副本集群上单独执行DDL操作。

**4.读取副本与最终一致性**
只读节点（或时间线一致节点）仍然严格优于最终一致性，因为对于后者，应用程序的数据视图可以及时来回移动并且难以编程。


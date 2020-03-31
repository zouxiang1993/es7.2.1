## ClusterState 
表示集群的当前状态。  
ClusterState对象是不可变的，除了routingNodes这个字段。   
ClusterState只能在master节点上更新，所有的更新均在单个线程上执行，由ClusterService控制。  
每次更新后，Discovery.publish()方法会向集群中的其他所有节点发布新版本的ClusterState。  
ClusterState实现了Diffable接口，支持以增量形式发布集群状态的更新。为了确保将差异应用到正确的ClusterState版本，每个集群状态版本更新都会生stateUUID来唯一标识此状态版本。

## 索引，分片，副本
一个索引(index)由多个分片(shard)组成，索引中的数据根据特定的哈希算法均匀分配到各个分片中。

一个分片就相当于pacificA算法中的一个副本组，有一个主副本和多个从副本组成。

分片——ES的分布式方案  
副本——ES的高可用方案


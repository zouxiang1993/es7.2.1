## ClusterState 
表示集群的当前状态。  
ClusterState对象是不可变的，除了RoutingNodes结构是根据RoutingTable来构建 ?   
ClusterState只能在master节点上更新，所有的更新均在单个线程上执行，由ClusterService控制。  
每次更新后，Discovery.publish()方法会向集群中的其他所有节点发布新版本的ClusterState。  
ClusterState实现了Diffable接口，支持以增量形式发布集群状态的更新。为了确保将差异应用到正确的ClusterState版本，每个集群状态版本更新都会生stateUUID来唯一标识此状态版本。
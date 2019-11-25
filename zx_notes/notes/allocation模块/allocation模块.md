参考资料: https://cloud.tencent.com/developer/article/1334740  

ShardsAllocator类 是在ES集群中的节点上进行分片分配的入口类。
如果一个已经分配的分片实例需要重新分配(由于节点故障，或者由于再平衡决定 rebalancing decisions)，
则由ShardsAllocator来决定这个分片要被分配到哪个节点上去。 

### BalancedShardsAllocator
基于BalancedShardsAllocator.WeightFunction来重新平衡集群中节点的分配。
集群平衡由下列参数定义，均可以实时更改: 
1. cluster.routing.allocation.balance.shard
2. cluster.routing.allocation.balance.index
3. cluster.routing.allocation.balance.threshold

这些参数组合在WeightFunction中，计算节点的权重，然后用于分片的再平衡。

BalancedShardsAllocator.allocate()流程: 
```text
// 分配尚未分配的分片
balancer.allocateUnassigned();
// 移动不能再停留在当前节点的分片
balancer.moveShards(); 
// 再平衡 
balancer.balance();
```


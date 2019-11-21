Discovery模块负责 发现节点、选举主节点、形成集群、发布集群状态。
它与其他模块集成，例如，通过transport模块来完成节点之间的通信。

这个模块分成以下部分:  
## 节点发现
"节点发现"是一个节点寻找其他能够与之形成集群的节点的过程。
当你启动一个Elasticsearch节点，或者一个节点认为master节点挂掉的时候，
"节点发现"过程就会开始，直到它恢复了与master节点的通信或者选出一个新的master为止。

此过程从以下两部分主机地址开始: 
- 由一个或多个seed hosts provider提供的一个种子地址列表
- 上一个已知集群中的所有master-eligible节点

该过程分为两个阶段: 
1. 节点会探测种子地址 —— 与每个地址通信，并尝试识别该节点，判断其是否是master-eligible节点。
2. 如果是。节点会将其已知的所有master-eligible节点共享给这个远程节点，同样的，
远程节点也会将它已知的master-eligible节点共享。然后节点会探测新发现到的节点，
请求新节点上的master-eligible数据。 

如果一个节点不是master-eligible节点，它会一直重复"节点发现"过程，直到它发现一个master节点。
如果未发现master节点，那么它会在1秒后重试，这个时间可以通过 discovery.find_peers_interval 配置。

如果一个节点是master-eligible节点, 它也一直重复"节点发现"过程，直到它发现了一个选定的主节点，
或者它找到了足够多的无主的master-eligible节点来完成选举。如果这二者都没有足够快的发生, 
该节点会在1秒后重试， 这个时间可以通过 discovery.find_peers_interval 配置。


## Quorum-based decision making 
选举主节点 和 更改集群状态 是所有master-eligible节点必须合作完成的任务。
即使某些节点发生故障，这些活动也必须能够正常运行，这一点很重要。

在ES中，每个(集群状态变更)操作都必须收到过半(quorum) master-eligible节点的确认之后才算成功。 
这种只需要部分节点响应的方式的优点在于: 在某些节点发生故障的情况下，集群仍然可以正常工作。
quorum的值是经过仔细选择的，因此集群不会出现"脑裂"的情况。

Elasticsearch允许您向正在运行的集群添加/删除 master-eligible节点。 
在许多情况下，您可以简单地通过启动或停止节点来执行此操作。

随着节点的添加或删除，ES通过更新集群的"表决配置"(voting configuration)来保持最佳的容错水平，
该配置是一个master-eligible节点的集合，在做决定(选举新master、提交一个新的集群状态)时，
只有该集合中的节点的响应才有效。

只有当"表决配置"中过半的节点做出响应以后，才能做出一个决定。
通常，"表决配置"就是当前集群中所有的master-eligible节点。但是，在某些情况下可能会有所不同。

为确保集群可用，你不能同时停止"表决配置"中一半或者更多的节点。
只要有一半以上的表决节点可用，集群就仍可以正常工作。
这意味着，如果有3-4个master-eligible节点，那么集群可以容忍其中一个节点不可用。
如果有两个或更少的master-eligible节点，那它们必须都保持可用。 

当一个节点加入或者离开集群后，当前的master节点会发布一条集群状态更新，用来调整"表决配置"以使其匹配，
这可能很快就能完成。重要的是，要等这个调整完成后，再从集群中删除更多节点。

### 选举主节点
无论是在启动时，还是在当前master故障时，ES都会通过选举过程来协商选举一个master。
任何master-eligible节点都可以发起选举，通常第一个发起的会胜出。
一般情况下，只有在两个节点几乎同时开始选举，才有可能会失败。
因此在每个节点上随机选择发起选举的时机以降低这种失败的可能性。
节点将不断重试，直到选举出一个master节点为止。 

### 集群维护，滚动重启和迁移
许多集群维护任务需要暂时关闭一个或多个节点，然后再次启动它们。默认情况下，
如果一个master-eligible节点下线(例如在滚动重启期间)，ES仍然可以保持可用。 
此外，如果多个节点停止后又重新启动(例如在集群完全重启期间)，ES也能自动恢复。
在这些情况下，无需采取任何进一步的措施，因为主节点集合不会永久改变。 

## 表决配置 (Voting configurations)
每个ES集群都有一个"表决配置", 这是一组master-eligible节点。
在做出决策时，只有"表决配置"中的节点的响应才有效。只有"表决配置"中的过半节点响应以后才会做出决策。

在节点加入或离开集群后，ES会通过自动修改"表决配置"来确保集群尽可能的具有弹性。 
在删除更多的节点之前，请务必等待此调整完成。

当前的"表决配置"存储在集群状态中，可以通过以下方式查看: 
```text
GET /_cluster/state?filter_path=metadata.cluster_coordination.last_committed_config
```

你可以通过 cluster.auto_shrink_voting_configuration 设置 来控制"表决配置"是否自动收缩。

如果将cluster.auto_shrink_voting_configuration设置为true, 并且集群中至少有3个节点，
则只要其中一个节点是健康的，ES都可以处理集群状态更新。 ??????

在某些情况下，ES可能要容忍多个节点的丢失，但这并不能在所有故障序列下得到保证。
如果cluster.auto_shrink_voting_configuration设置为false, 则必须手动从"表决配置"中删除离开的节点。
使用  voting exclusions API 来达到所需的弹性水平。 

### 奇数个master-eligible节点
集群中通常应该有奇数个master-eligible节点。如果有偶数个，ES会将其中一个排除在"表决配置"之外，
以确保其大小为奇数。
这个省略不会降低集群的容错能力。相反的，还稍微提升了集群的容错能力：
如果集群遇到网络分区，将其分成大小相等的两个部分，则其中的一个部分将包含过半的"表决配置"，并且能够继续运行。
如果没有排除掉这一个节点，那么两部分都不会包含过半节点，因此集群无法继续运行。 

### 设置初始的"表决配置" 
当一个崭新的集群首次启动时，要选出第一个master节点，就必须知道"表决配置"。
初始的"表决配置"在节点的配置文件中进行设置。 

仅在集群首次启动时才需要这个配置。新加入集群的节点可以安全的从master节点获取所有信息。
先前属于集群的节点已经将这些信息存储在data文件夹下，可以在节点重启或者整个集群重启时使用。

## 添加和移除节点 
建议在集群中拥有少量且固定的master-eligible节点，并且通过添加/删除 非主备节点 来进行集群的扩缩容。
然而，在某些情况下，可能还是要添加/移除 master-eligible节点。

### 添加master-eligible节点
如果要向集群中添加一些节点，只需要配置新节点使之能够找到现有的集群，并启动节点。
ES会自动将新节点中合适的节点添加到"表决配置"中。 

在选主期间，或者在一个节点加入一个已有集群的时候，节点会向master节点发送加入请求。
你可以使用cluster.join.timeout来设置节点在发送加入请求之后等待多长时间。默认30秒。

### 移除master-eligible节点
在移除master-eligible节点时，重要的是不能同时移除太多。例如，如果当前有7个master-eligible节点节点，
你希望减少到3个，则不可能简单的一次性停止4个节点：这样做将只剩下3个节点，少于一半的"表决配置",
这会导致集群无法工作。如果发生了这种情况，只能通过重启已经移除的节点来使集群恢复可用。 

如果集群中至少有3个master-eligible节点，通常的做法是一次只移除一个节点，以保证集群有足够的时间来自动调整"表决配置"。

但是如果集群中只有2个master-eligible节点，则不能按期的删除任何一个节点。
想要删除其中一个，你必须先通知ES它不应该是"表决配置"中的一部分，然后再移除节点。
 
可以通过 Voting Configuration Exclusions API 来操作: 
```text
# Add node to voting configuration exclusions list and wait for the system
# to auto-reconfigure the node out of the voting configuration up to the
# default timeout of 30 seconds
POST /_cluster/voting_config_exclusions/node_name

# Add node to voting configuration exclusions list and wait for
# auto-reconfiguration up to one minute
POST /_cluster/voting_config_exclusions/node_name?timeout=1m

# Wait for all the nodes with voting configuration exclusions to be removed from
# the cluster and then remove all the exclusions, allowing any node to return to
# the voting configuration in the future.
DELETE /_cluster/voting_config_exclusions

# Immediately remove all the voting configuration exclusions, allowing any node
# to return to the voting configuration in the future.
DELETE /_cluster/voting_config_exclusions?wait_for_removal=false
```

## 发布集群状态
master节点是集群中唯一可以更改集群状态的节点。master节点一次处理一批集群状态更新，
计算所需的更改并将更新后的集群状态发布到集群中的所有其他节点。每个发布都从master节点开始，
向集群中所有节点广播更新的集群状态。每个节点都会回复一个确认响应，但尚未应用新接收的状态。
一旦master从足够多的master-eligible节点收到了确认，就称新的集群状态已提交，
master节点会广播另外一条消息，指示所有的节点应用当前已提交的状态。
每个节点收到消息后应用新的集群状态，并向master节点发送第二个确认消息。

master上每一次集群状态更新并发布到所有节点都是有时间限制的，它由cluster.publish.timeout定义，默认为30秒。
如果在新集群状态提交之前，时间到了，那么master会认为自己失败了。它退位下来，并开始新一轮的选主。

如果在cluster.publish.timeout耗尽之前提交了新的集群状态，则主节点认为更改已经成功。
它等待，直到超时或者它收到集群中每个节点已经应用更新状态的确认消息，
然后才会开始处理并发布下一个集群状态更新。如果尚未收到某些确认，则称这些节点为滞后状态，
因为他们的集群状态已经落后于主节点的最新状态。master会再等待这些滞后节点一段时间，
即cluster.follower_lag.timeout，默认90秒。如果节点在此时间内仍未成功应用集群状态更新，
则认为该节点已经失败并将其从集群中删除。   

## 集群故障检测
master节点会周期性的检查集群中每个节点以确保联通和健康，称为leader checks。
其他节点也会周期性的检查master节点，称为follower checks。

ES允许这些检查偶尔失败或超时，而无需采取任何措施。仅在许多连续检查失败后，它才认为节点有故障。
你可以使用cluster.fault_detection.* 设置来控制故障检测的行为。  

但是，如果master节点检测到节点已断开连接，将这种情况视为立即失败。master节点尝试将该节点移出集群。
同样，如果节点检测到master节点断开连接，它会重新启动它的"节点发现"阶段去尝试发现或选举一个新的master














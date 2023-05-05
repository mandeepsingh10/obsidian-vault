### What is ETCD?

- ETCD is a distributed reliable **key-value store** that is simple, fast and secure.
- ETCD service runs on port **2379**
- In case of multiple ETCD instances in a HA environment, we can write to any instance and read the data from any instance.
- Internally in an HA environment only one ETCD instance is responsible for write operations , one node becomes the leader and other nodes becomes the followers.
- If the write comes through the leader node then the leader processes the write, the leader amkes sure that the othe nodes are sent a copy of the data.
- If the writes came in through any of the other follower nodes, then the followers forward the write operations internally to the leader node and then the leader processes the write.
- A write operation is considered to be complete only when it is replicated to the other nodes in the ETCD cluster.
- A write operation is considered to be complete if it can be written on the majority of the nodes in the ETCD cluster., suppose we have 3 nodes in the cluster, if the data can be written on 2 of the nodes then the write operation is considered to be complete.

### QUORUM
- A more appropriate term for majority would be QUORUM, it is minimum numbe rof nodes that must be available for the cluster to function properly or make a scuccessful write, for a 3 nodes, the QUORUM=N/2 +1 i.e 3/2 +1 = 2.5 ~=2.
- If the resulting number is 2.5 then consider the whole number only i.e 2.
- QUORUM of 1 is 1 itself, meaning if you have a single node cluster, none of these apply.
- Incase of 2 node cluster, the QUORUM is 2, that means if a node fails then the writes won't be processed, so having two instances is like having 1 instance so it doesn't provide any real value as QUORUM cannot be met, which is why it is recommended to have minimum of 3 nodes in an ETCD cluster.
- So minimum number of nodes for setting up HA ETCD cluster is 3.

###  Fault Tolerance

The number of nodes in the cluster - the QUORUM gives us the fault tolerance i.r the number of nodes we can afford to lose while keeping the cluster alive.

EXAMPLE:
Number of nodes = 5
QUORUM = 5/2+1 = 3.5 ~=3
Fault tolerance => 5-3 = 2

This means that in a 5 node cluster, the cluster can function even if 2 nodes fail.

### Deciding on the number of Master Nodes for ETCD

- With even number fo nodes there is a possibility of cluster failure in case of network segmentation, so we should choose odd number of nodes for an ETCD cluster.
- Suppose we have a 6 node cluster, for some disruption in the network it fails and causes the network to partition, we now have 4 node on one partition and 2 on the other parition.
- In this case the the partition with 4 nodes continues to work normally because it has QUORUM but the parition with 2 nodes will fail but we will still have a working cluster because QUORUM is met for the 4 node partition.
- But in case the network segmentation happens in a way in which it partitions the cluster into two groups of 3 nodes then we won't have a working cluster because initially in a 6 node cluster the QUORUM was 4 but neither of these two partitioned group has QUORUM as 4 so our ETCD cluster will fail.
- With Odd number of nodes, no matter how the network segmentation is done, we will always have atleast one group satisfying the QUORUM so our cluster will function.



### ETCDCTL Commands


# Deterministic partitioning

To achieve our size and throughput goals we need to 
find a partitioning scheme with the following properties:

- Deterministic: all the observers should agree to the same
partitioning with minimal information exchange
- [Consistent](https://en.wikipedia.org/wiki/Consistent_hashing): 
removal or addition of nodes should cause minimal reshuffling
of keys
- Hierarchical: At scale it's best to organize our 
nodes in trees, thus introducing a concept of structures 

Moreover, we observe that while the addition of new nodes can
be planned and executed in a deterministic way, the failure of
a node can happen at any time, and thus should be the case for
which we optimize for.

[Rendezvous hashing](https://en.wikipedia.org/wiki/Rendezvous_hashing)
is a generalization of the consistent hashing algorithm used 
by dynamoDb, which allows to choose `k` nodes in a consistent
way, with minimal information sharing across observers. 

Rendezvous hashing alone is not enough to achieve our goals, as in 
the naif implementation node removal or addition would require 
that all nodes know about each other, which is too expensive
for large clusters. We need to introduce a concept of 
[hierarchical structures](https://en.wikipedia.org/wiki/Rendezvous_hashing#Skeleton-based_variant),
which is a way to partition the nodes into groups, which can be
used to achieve the desired properties.

# Hierarchical Structures

The skeleton based variant of the rendezvous hashing algorithm gives
us the idea of building a natural hierarchy, to build a tree which
will speed up searches. Before doing that, let's define 
which are the failure modes of the cluster:

- The smallest possibility is disk failure.
- Then we have node failure, when all the nodes to which a group of disks
is attached dies.
- Then we have several kind of group failures: rack failure, room failure,
datacenter failure, and so on.

We expect the disk failure to be the most common failure mode, and the one
for which we will have the highest replication factor. This allows us
to define a natural hierarchy:

- Regions
- Zones
- Groups
- Nodes
- Disks

Which we can extend for performance reasons:

- Namespaces
- Disks Queues

## Disk queues

Ensuring that a key is deterministically assigned to a queue in 
one of the replicas increase for free the consistency of the system:
in case a weak read starts while a consistent write is still in progress,
we will get for free the correct value, because the NVMe command will 
not resolve until the flush of the write is done.

# Failure example

For each hierarchy level there is a quorum: let's say a group of 24
disks are attached to 2 nodes. If a disk fails, the system will
rebalance the load across the remaining 23 disks without problems. 
If instead both nodes have a power failure, the system will trigger
a rebalance across nodes. According to our availability goals, we can tolerate a number 
of failures per level before causing a rebalance at a higher hierarchy.

# Scaling example

Generally using weights with the rendezvous hashing algorithm is not
a good idea, because changing the weights will cause a rebalance across
many keys. 

Instead, we will use weights to plan for future expansion of the cluster,
which can be planned in advance. We can create fake zones or nodes and 
assign them a weight of 0. When we start adding new nodes, we can assign
a weight of 1 only to new accounts, to allow for gradually scaling up.
Eventually all the accounts will see a weight of 1, but the rebalancing
will be way more gradual.

See an example [here](examples.md). For an implementation please see [here]()

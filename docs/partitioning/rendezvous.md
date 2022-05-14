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

## Hierarchical Structures

Quorum at different levels of the hierarchy can be achieved by 

NVMe queues deterministically.

For an implementation please see [here]()

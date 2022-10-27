# The problem

To achieve our size and throughput goals we need to
find a partitioning scheme with the following properties:

- Deterministic: all the observers should agree to the same
  partitioning with minimal information exchange
- [Consistent](https://en.wikipedia.org/wiki/Consistent_hashing):
  removal or addition of nodes should cause minimal reshuffling
  of keys
- Hierarchical: the scheme is sensitive to locality

To solve this we use [rendezvous hashing](https://en.wikipedia.org/wiki/Rendezvous_hashing), 
 which allows us to choose `k` nodes in a consistent way, with 
minimal information sharing across observers.

This alone is not enough to solve all our problems: a
naif implementation  would require that all nodes to know about each 
other status, which is too expensive for large clusters. 
We need to introduce a concept of **hierarchicy tree**,
which is a locality sensitive way to partition the nodes.

Moreover, we observe that while the addition of new nodes can
be planned and executed in a optimal way, the failure of
a node can happen at any time, and thus should be the case for
which we optimize for.

# Hierarchicy tree

The [skeleton based variant](https://en.wikipedia.org/wiki/Rendezvous_hashing#Skeleton-based_variant)
of the rendezvous hashing algorithm gives us the idea of building 
a tree to speed up operations. 
But before doing that, let's analyze the failure modes of the cluster:

- Under the assumption that our code is correct we can assume 
that shards will not fail. In practice we can recover even from that.
- The smallest mode of which we have no control is the disk failure.
- Then we have node failure, where all the disks attached to a node become unavailable.
- Then we have several kind of group failures: rack failure, room failure,
  datacenter failure, and so on.

We expect the disk failure to be the most common mode, and the one
for which we will optimize, and then the other modes in a decreasing 
order of probability. This allows us to define a natural tree-like 
hierarchy based on failure modes, for which the levels are:

- Regions
- Zones
- Racks
- Nodes
- Disks
- Shards



# Introduction

## Tree Layers

- Region: Picked ad config time
- Zone: DC, or group of closely related DCs
- L1,...Ln: Skeleton construction
- Node: Node, or group of closely related nodes
- Shard: Namespace, or group of closely related namespaces
- Blocks: List of blocks containing data


## Walk the tree

Perform rendezvous at each layer of the tree, inspired by the 
skeleton construction

## Collection step

For each collection and node type (random, linear, compute):

- `R`: Replication factor
- `MAX_LOAD`: Maximum number of shards from the same
    collection in a node, power of 2
- `S = N * MAX_LOAD`: Shard factor, greater or equal than `MAX_LOAD`
- `N`: Number of nodes, power of 2


- Pick regions at config time for each collection
- Pick `R` walks starting at the root of the tree
- At the next level of the tree, pick `N` walks
until you reach a node
- (Optional): Add `log(N)` walks, to pick failover nodes
- For each node pick `MAX_LOAD` shards

We end up with a pool of `R * S` shards, distributed among `N` nodes
in `R` regions/zones. We also have a pool of `R * log(N)` nodes as
backup.

## Record step

For each record in a collection, given a pool of `R*S` shards

- Start `R` walks
- For each walk, pick one node
- For each node, pick one shard

You end up with `R` shards, which should all contain 
replicas of your record

## 

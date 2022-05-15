# Key addressing

Let's say we have a 128 nodes spread across 2 regions and we want
to know the position of a key: 

`account/tableName/resourceType/primaryKey`

We know that the replication factor is 6, and the partitioning factor
is 16. We decide to store the data in a single region.

Our hierarchy could be:

- 2 Regions
- 2 Zone per region
- Each zone has 32 nodes with 16 disks each.
- Each disk has 8 1TB namespaces,
- Each namespace has 128 queues

This means that we have a total of 2^21 combinations. Using a naif rendezvous hashing 
would be too expensive. We start by understanding which nodes are responsible for the 
table. Given the root of the key: `account/tableName/resourceType`, we need to pick
16*6 = 96 nodes using the rendezvous hashing on this tree:

- 2 Regions, with weight [1, 0]
- 8 Zones, with weight [1, 1, 0, 0, 0, 0, 0, 0] 2
- Nodes, without weight
- Disks, without weight
- Queues, without weight

The first level is region, the weights forces the algorithm to pick the first region, 
to express a user preference and doesn't need hashing.
The second level is zone, with 2 real zones and 6 fake zones to plan for future expansion. 
Given that we need 96 nodes, we know that both zones will be involved.
The third level is node, with 32 possibilities times 2 zones. For each zone we will perform
a rendezvous hashing, and pick 16 nodes.
The fourth level is disk, with 8 possibilities times 16 nodes.

We have thus selected 256 possibilities. We can pick the 96 disks with the highest value and thus
have our pool. We can now build a tree which represent the `account/tableName/resourceType` 
hierarchy:

- 1 region
- 2 zones
- 16 nodes
- 16 disks per node
- 8 namespaces per disk

Now everytime we want to know the position of a key for the table, instead of performing another round 
on the cluster tree, we can do a lookup on this smaller tree:

- 2 zones
- 16 nodes
- 16 disks per node
- 8 namespaces per disk
- 128 queues per namespace

For a total of 2^19 possibilities. We can build a virtual tree:

- 32 virtual nodes
- 32 virtual nodes
- 32 virtual nodes
- 16 queues

To know the position of a key we need 4 steps, 8 steps if we want to know all the 6 replicas.










# Gossip agreement protocol

The GAP is the protocol tasked with building the hierarchy tree.


# Optimizations

The traditional rendezvous gossip protocol relies on picking a random node 
for each round of gossip. This is means that unless for lucky choices 
we need more than `o(log(n))` steps to converge. Please see our 
simulations.

Here we propose a couple of simple improvements which we call rendezvous 
gossip, with the following improvements:

- Guaranteed `o(log(n))` convergence
- Fast failure detection
- Very large cluster size (10^9 nodes)

Implementation [here]().

## Small cluster

// TODO: put in rebar

Let's say we have a cluster of 8 nodes and the fanout is 3. 
The cluster is already at equilibrium, so each node knows about all the other nodes. 

For the next round, each node perform a round of rendezvous hashing to determine 
which node to send the next message to. The hashed string is:

`nodeName,iteration,peer1,peer2, ... peer8` 

Which is built using the lexicographic order of the node hostnames. The 512 bit 
hash is split in 8 chunks of 64 bits each, and the next 3 nodes are selected.

The message is sent over the UDP protocol, and contains a hash of current 
known list of healthy nodes:

`peer1,peer2, ... peer8`

Each peer can then immediately verify if all the senders agree on the list
of peers. If not then a synchronization over TCP is performed.

This approach guarantees that in `o(log(n))` steps not only the network
will converge, but also that each node will also receive at least
one confirmation, for extremely fast failure detection.

## Large clusters

Up to 1024 nodes the above strategy is fine, but failures becomes
increasingly frequent and more expensive as they will 
require a lot of synchronizations.

To scale up we can start using hierarchies and merkle trees:

- Nodes are organized in groups
- Groups are organized in Zones
- Groups can have a size from 25 nodes to 250 nodes
- Each node knows about the local nodes, and the hash of other groups. This control the size of the message.
- Only few nodes communicate across groups

Let's say we have a group of 25 nodes, and a fanout of 6. We know that there are 4 groups in 2 zones.
For a node in group 1, our string thus becomes:

`peer1,peer2, ...,peer25,group2`

When any node select group2 as a candidate, it will send the message one node in group2. 
The message is generated using a merkle tree.




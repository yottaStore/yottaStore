# Readme

# Architecture

- Yottastore: compute layer
- Yottafs: storage layer
- Client


# Modules

- Yottadb: module for yottastore
- Self: awareness and optimizations, ml based
- Rendezvous: module for yottastore
- Gossip: module for yottastore and yottadb
- Yottapack: optimal serialization format


## Yottadb

Adds advanced features to yottastore. 
Possible drivers are:

- KeyValue: simplest one, default
- Columnar: Column values
- Document: For collections
- PubSub: optimized for message queues
- Indexed: support queries, based on btrees
- Graph: optimized for trees and graphs

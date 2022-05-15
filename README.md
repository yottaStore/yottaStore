# Introduction
Yotta Store is a storage system aiming at **linear scalability up to the yotta byte range**. 
The goal is to have **two orders of magnitude more throughput than DynamoDB**, 
dollar per dollar, while maintaining a **sub-ms latency**. 
Check our [benchmarks](docs/10_benchmarks/benchmarks.md)

Yotta Store is built on top of a 512 bit distributed machine, with a large word size.
We try to design a system which can exploit the capabilities of
modern hardware and software, like  NVMe disks or the Linux kernel. 
Read more in the [docs](docs/README.md)

## Main features

- Linear scalability, up to 10^9 nodes and 10 yotta bytes of addressable space.
- Anti fragility, the multi tenant setup increase reliability and availability with load.
- Strong consistency guarantees, aiming at sub ms latency.
- Cheap transactions and indexes, at around `o(n)`.
- Storage decoupled from compute with a serverless architecture.
- Two orders of magnitude faster than DynamoDB, dollar per dollar.


## Techniques used

Yotta Store does not create any new technique, but uses existing ones in a novel combination:

| Problem                          | Solution                                                                            | Description                                                                                                               |
|----------------------------------|-------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Partitioning and replication     | Rendezvous Hashing                                                                  | A generalization of consistent hashing, for agreeing k <br/>choices in a stable way and with minimal information exchange |
| Highly available writes          | Compare and swap atomics, version vectors, <br/>conflict-free replicated data types | hhh                                                                                                                       |
| Transactions                     | Warp algorithm                                                                      | hhh                                                                                                                       |
| Indexes, queries across keys     | Concurrent linked lists, partitioned indexes                                        | Indices are lazily built by subscribing to key changes                                                                    |
| Membership and failure detection | Rendezvous gossip                                                                   | Thanks to rendezvous hashing and merkle trees the expected cost is `o(log(n))` deterministically.                         |


## Inspirations

- [Designing Data-Intensive Applications, Martin Kleppmann](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/)
- [The Dynamo paper, many authors](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [A pragmatic implementation of non-blocking linked-lists, Timothy L. Harris](https://timharris.uk/papers/2001-disc.pdf)
- [Warp: Lightweight Multi-Key Transactions for Key-Value Stores, many authors](https://arxiv.org/pdf/1509.07815.pdf)
- Kafka



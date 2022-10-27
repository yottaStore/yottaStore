# Documentation

I hope that the reader that will bear with reading this, will then be able to fully understand the system, 
and implement one himself if he decides so.

### Chapter 1: Architecture

- [Chapter 1.1](01_architecture/11_problem.md): The problem
- [Chapter 1.2](01_architecture/21_principles.md): Design principles
- [Chapter 1.3](01_architecture/31_machine.md): The distributed machine
  - [Chapter 1.3.1](01_architecture/32_instructions.md): The instruction set
- [Chapter 1.4](01_architecture/41_consistency.md): Consistency Model


### Chapter 2: Leaderless partitioning and replication

- [Chapter 2.1](03_partitioning/11_introduction.md): Introduction
- [Chapter 2.2](03_partitioning/21_rebar.md): Rendezvous based routing
- [Chapter 2.3](03_partitioning/31_gap.md): Gossip agreement protocol
- [Chapter 2.4] Examples
  - [Chapter 2.4.1](03_partitioning/41_examples.md): Partitions and replicas
  - [Chapter 2.4.2](03_partitioning/41_examples.md): Key routing
  - [Chapter 2.4.3](03_partitioning/41_examples.md): Gossip rounds


### Chapter 3: YottaFS

- [Chapter 3.1](05_yottafs/nvme.md): The best filesystem
- [Chapter 3.2](05_yottafs/warp.md): Transaction and indexes

### Chapter 4: Infrastructure

- [Chapter 4.1](07_infrastructure/11_serverless.md): Decoupling storage and compute
- [Chapter 4.2](07_infrastructure/12_hybrid.md): Hybrid cloud and standby replicas
- [Chapter 4.3](07_infrastructure/13_sizing.md): The best node size (queue theory)

### Chapter 5: Benchmarking

- [Chapter 5.1](10_benchmarks/benchmarks.md): Performance and costs estimates


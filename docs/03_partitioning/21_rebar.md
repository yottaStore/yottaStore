# Rendezvous based routing alogrithm

The Rendezvous Based Routing (ReBaR) algorithm
is used to navigate a hierarchy tree, to find
the shard responsible for a record. It's called
like that because it uses a rendezvous hashing
approach to agree on a set of choices with 
minimal information exchange. 

Given a record in the form:

`account@driver:collection/record/subrecord`

And a hierarchy tree, how do we find the 
shards responsible?

Before answering that, we need the user to tell us:
- The region(s) involved
- The type of replication (single region or multi region)
- The sharding factor `s`
- The replication factor `r`


In total, we will need to find `s*r` shards, spread across
one or multiple regions according to the replication type.
To do that we split the algorithm in 3 steps:
- Parsing the record
- Finding the shard pool responsible for the collection
- Finding the shards responsible for the record from the pool


## Parsing the record

Given a record string, it is parsed in a struct like:

```go
package main

type Record struct {
	Account string
	Driver string
	Collection string
	Record string
	PoolPtr string
	ShardPtr string
}
```

// TODO: put link to driver types
The driver [type]() 
will determine which types of nodes are involved
in the collection (`linear` or `random` nodes).

The `PoolPtr` is in the form `account@driver:collection` and
is the same for all the records in a collection.

The `ShardPtr` is in the form `account@driver:collection/record` and
is the same for all the subrecords of a root record.

## Hashing step

Before describing the next parts, let's define the hashing step, 
which is a slight variation of the classic rendezvous hashing.
Given a 

## Finding the pool

Each collection is assigned to a pool of shards, determined by
the `PoolPtr` of the record. This has the advantage that is 
possible to enumerate all the shards of a collection, making queries easier.

We start by picking a region

## Finding the shard

Once the pool is found, we need to find the shards responsible for the record

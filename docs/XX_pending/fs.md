# A pragmatic implementation of Wait Free Persistent Storage

## Abstract

We present a pragmatic implementation of a wait free persistent storage, designed
around NVMe characteristics and asynchronous IO with `io_uring`. 



## Introduction

In the context of distributed datastores latency and predictability are key metrics, 
with implications on the transactional throughput and the cost of the system.


## Constraints

## Related work

## The algorithm

The fundamental data structure of the algorithm is the Record:

```

const (
    Size = 4096 // Underlying block size
)
type Block [Size]byte

type Record struct {
	Body []Block
	Tails [][]Block
	Appends []
}

```

## XFS implementation

The fundamental structure is a 


## NVMe implementation

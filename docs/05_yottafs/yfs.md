# Yottastore File System

User space filesystem. Shard-per-core, single threaded concurrent. Parallelism is achieved by sharding. 

Listen on Unix Domain Socket to requests from network demon.

Garbage collector and compactification done concurrently on separate thread (see 
[Trim](./trim.md))

Have big hashmap (hash trie?) in memory, pointing uints to sectors. (resize?)
If hashmap is 32 bit key + 32 bit value, then 2 gb every TB of SSD.
Compute node takes care of hashing and issuing consecutive commands.

## Loop

For loop, wait for io_uring events.

In case of request (socket event), issue the command with UUID (32 bit unique, + 32 bit flag) 
and store it in hashmap.

In case of IO ready, use UUID to find relevant request and issue send command (on socket).

# Operations supported by filesystem

- Read
  - Read Range
  - Read Follow
- Write
- Append
- Delete
- Compare
- Compare and Swap
- Flush
- Verify


## Read

Read take a `uint`, and return the sector which the hash trie points to.

Read Range takes a list of ranges to read

Read Follow return the sector, and any sector the skip list points to.


## Write

Write attempt to WAL.
Write a data stream to disk, for the given key. Takes care of using multiple sectors if needed.
If key exist, it gets rewritten. Compute node is responsible for merging existing data.
If successful, record to WAL and return true.



Options:
- Compressed: signal that compute node compressed the data
- Compact: if size is much less than sector, signal that compactification with other records is desired
- Revert: possible to mark the write as revertable
- Compare: verify existing state before writing, fail if mismatch
- Flush: Make write strongly consistent

## Append

Append a data stream to an existing key. Takes care of using multiple sectors if needed.
Useful for columnar records.

Options:
- compressed: signal that compute node compressed the data
- compact: if size is much less than sector, signal that compactification with other records is desired
- Revert: possible to mark the write as revertable
- Compare: verify existing state before writing, fail if mismatch
- Flush: Make append strongly consistent

## Delete

Given a key, mark it as deleted.

If zone is full, compute share of deleted sectors per zone, if it is above threeshold then issue trim command.

## Compare

Verify that a key has not changed sector pointer in the hashmap.

Return true or false.

## Compare and swap

Write temp data.
Verify that a key has not changed sector pointer in the hashmap.
If not, perform a swap.

Possible to be multi word, to support transactions.

Return true or false.

Optional: 

Count failures through sliding window bloom filter, to switch from CAS to Locks for high contention.

## Flush

Issue Flush command to empty the write queue.

## Verify

Given a key, verify that it matches the CRC. Done by the compute node.

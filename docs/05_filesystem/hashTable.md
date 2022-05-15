# Hash Table

Here we describe a hash table optimized for NVMe storage. For 
simplicity, we assume a namespace of 1 TB, and a block size of 4 KB, 
which means 2^28 blocks.

If we were to implement a naif hash table, we would face the following
challenges:

- Poorly utilized space
- Needs wear leveling
- Values bigger than a block
- Needs to handle collisions

## Pointers array

When we hash the key, we obtain a value which represents the index of an array. 
The array at the index will contain either the value of the key or a 
pointer to the value:

- 512 bits to reference the key hash
- 32 bits to reference the first block containing the value
- 16 bits to represent how many blocks are used to store the value

Even if we assume a total size of 1024 bits, this means a 4 KiB block
can contain 32 pointers. Let's say we decide to store 8 pointers per block.

We allocate a large numbers of blocks to store the buckets of the hash table, let's say
64 gb or 2^24 blocks. This means we can address a total of 2^27 keys, or 512 GiB at 
4 KiB per key. The remaining space on the device is used to store the values, 
which can be of arbitrary size.

To further improve wear leveling, we decide to aggregate 3 blocks together, and rotate writing:
Each block will contain up to 24 pointers, still less than the maximum of 32 pointers,
and everytime a record is created or updated, we will rotate the block written.

A record key is created or updated less often than the value, so this approach should
provide optimal performance and endurance. We now need a way to efficiently
allocate memory pointers.

## Memory allocation

The first 128 blocks are reserved for the disk metadata, the following 64 GiB are 
reserved for the associative array, this means we have around 960 GiB of space to
store the values.

We need a mechanism to keep track of which blocks are free and which are used. 
To do that we can store a list containing the free memory pointers, and the number of blocks.
For example at the beginning it will be:

`[{0: 2^30}]`

Which means starting at the address 0, we have 2^30 blocks, which are free.

Out of this space we decide to allocate 1024 blocks for keeping track of the free blocks, so
our free space will look like:

`[1024: 2^30-1024]` written at block 0 and 512.

Which means that the first free block is 1024, and we have 2^30-1024 free blocks.
When we want to save a new key, we can read the the first 1024 blocks, detect the most
recent state, and write the new state with a compare and swap. The atomicity of 
the compare and swap allow us concurrency without locks, for very high performance.

To avoid block trashing, we can decide to allocate a little bit more memory than required,
for example 30% more, rounded up. In case we need to write one block, we will allocate two,
so that successive updates can rotate the blocks without needing another allocation.

Smart allocation will be used, to ensure that the list will not grow unbounded.

# The distributed machine

Our approach to solve the task is to build a distributed machine, 
with a 512 bits memory pointer and a large word size.

To build that we need:

- Memory pointers 
- Memory to store data
- A virtual machine with an instruction set


## The memory pointer 

To access the memory we will use a hash table: given the key, it's possible to calculate 
the hash and thus deterministically access the value of the memory by determining which 
node is responsible for it. A key is of the form:

`account/resource/resourceType/primaryKey/sortKey`

- `account` to allow multi-tenancy on the same machine. 
- `resource` can be either the table name or the queue name
- `resourceType` can be either `rnd` for random access keys, backed by SSDs,
or `lnr` for queues, backed by HDDs.                                        
- `primaryKey` represents the key to which a value is associated. 
- `sortKey` is optional and is not hashed, like in dynamoDB. 
Typically used for versioning, it permits to store different values of 
the same primary key together.

The hash used is `SHA-512`, hence the 512 bit size of the pointer. To determine which 
node is responsible for the key, we use [rendezvous hashing]()

## The memory

To achieve our size goals, we will treat NVMe SSDs as our main memory. Although not as fast as RAM, 
SSDs provides excellent throughput and reliability. Having a non-volatile random memory means that is
much easier to recover from failures, while still providing exceptional performance characteristics.

More in [SSD vs RAM performace]()  

## Word size

The large word size is due to the nature of the backing memory. For random 
access is the SSD sector size which is 4 kb, even if a value is smaller than 
that the controller will still use a 4 kb sector, so it's better to optimize for that.

In the case of linear memory, which is typically an 
[SMR HDDs](https://en.wikipedia.org/wiki/Shingled_magnetic_recording) 
the word size is much bigger, up to 32 mb, to optimize for the 
high linear performance characteristics of hard disks. Queues
can be temporary stored in random memory before being aggregated
on disk, like the metadata needed to read the queue. More [here]()


## The virtual machine

For simplicity, we can take the WebAssembly virtual machine instruction set as reference,
with a few important additional instructions:

- LOAD $POINTER: Load the memory stored at POINTER
- COMPARE $POINTER $VALUE: 
- WRITE $POINTER $VALUE
- COMPARE&SWAP $POINTER $VALUE1 $VALUE2: Atomic compare and swap in the global memory 

These instructions are used to build lock free concurrent data structures, the key for the 
high availability of this system. For a more detailed discussion read [here]()


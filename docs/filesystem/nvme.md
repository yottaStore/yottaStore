# Filesystem 

It's possible to experiment with several different filesystem configurations,
among the most common ones are:

- **ext4**: Old, but reliable. 
- **BTRFS, NILFS**: Better suited for our case.
- **NOATIME**: To improve performance.
- **RAID**: to control replication and availability.
- **TRIM**: To improve latency

Instead I would recommend a different approach, more aligned with our hardware 
capabilities, which is to use the **NVMe** specification

# NVMe specification

NVMe is a new specification for storage devices. It gives use a set of commands
to access the SSD as a block device, without any filesystem. It exposes a set 
of basic operation like:

- **READ**: read a number of sectors on the device
- **WRITE**: write a number of sectors on the device
- **FLUSH**: execute all the commands in the queue
- **VERIFY**: verify the data on the device
- **COMPARE**: compare the data on the device with the data in the memory

On top of that we also have `fused operations`, which allow us to combine
a `compare` and `write` operation in a single atomic command. This is 
fundamental for our [instruction set](../architecture/instruction-set.md)

Discarding a filesystem gives us some new responsibilities:

- Detect and interface with the NVMe device
- Define a disk format
- Handle wear leveling
- Handle trimming
- Handle replication (RAID)
- Extend this to HDDs

Let's solve them one by one

## NVMe interface

Although powerful, the NVMe specification can feel daunting. Thankfully, 
there exist a very nice library tailored exactly to our needs:
[Storage Performance Development Kit](https://spdk.io/)

Behind a set of simple C primitives, it provides a high level of abstraction
to the NVMe device, while fully residing in user land. This allow us 
to exploit the high concurrency of the device, by setting up multiple queues
with high depth.

## Disk format

NVMe devices appears to Linux as a block device, which can be partitioned 
in several **namespaces**, backed by a number of underlying blocks. For 
simplicity of discussion, we will assume a namespace size of 1 TB, 
which at 4kb per sector, is equivalent to 2^38 blocks.

### Disk Metadata

The disk format is a set of metadata that describes the disk layout, and 
allow us to parse the contents of the disk. It's especially important
that this data is not corrupted, as it would make the disk unusable.

It's easy to imagine that a block of 4kb is more than enough to 
store the metadata, but we will reserve 128 blocks instead, and store
the initial metadata at position 0 and 64, as copies. It's easy to 
allocate 2 additional blocks, in case the metadata is bigger than 4kb.

If for any reason the metadata needs to be updated, we will store the
data in the next blocks, 1 and 65, again as copies. Once a few copies 
will be written, let's say 9, we will write a number of zeros to the 
oldest block, to mark it as available.

Whenever a process wants to read the metadata, it can simply ask for
the first 128 blocks of the namespace, and then parse the most recent
block. The double copy is to provide corruption resilience.

### Block format

Since we are dealing with a low level device, it's useful to define
a binary format for the blocks. We can take for example `grpc` as a 
 serialization format, maybe with a couple of addition:

- We need a pointer to the next block, in case one block is not enough
to store the data, and the next chunk is not in the same block.
- We need a checksum to verify the integrity of the data.

Read more here.

### Wear leveling

The metadata chapter give us an idea of how wear leveling can be implemented. The
application can decide to aggregate several writes in a single block, and then use
the free space to provide wear leveling. Ideally we would like to have a mechanism
to determine in advance which are the possible blocks, to submit the read to multiple
blocks as a single command.

### Trimming

Like with wear leveling, we can also implement a mechanism to trim the disk inspired 
from the metadata chapter. It seems we need a mechanism with the following properties:

- Keep track of the free blocks
- Trim the unused blocks in an asynchronous way
- Be thread safe

It seems like we need a non blocking garbage collector, which will be useful also 
for other scopes. More information here.

## Replication

Normally replication is handled both by RAID and the application, which is a waste.
We instead can implement replication at the application level, with several advantages:

- Unlike RAID, replicas can be read and write concurrently, increasing the throughput
- We can guarantee that copies will be spread across machines, increasing reliability
- Replicas are used as voting machines

The rendezvous hashing allow us to choose in a deterministic way the nodes
that will hold the replicas for our key, and thus chose accordingly.

## Hard disks

We can now extend the disk model to hard disks, which are optimizied for linear 
performance. If we want to publish to a queue, we can first determine which
nodes are the owner of the queue, and then publish to that node.

The problem is that a message is usually small in size, and we would prefer to write
32 mb chunks at a time. To achieve that we can use an intermediate cache on the 
SSDs: Let's say we allocate a chunk of 512 mb. Everytime a message arrives
we write a 4 kb chunk to the disk in the first free block. When enough blocks
are written they can sent to the hard disk. The large chunk allocation allows
for wear leveling, non blocking concurrency writes.

The only thing we need is to store the metadata of the disk, with the list of chunks 
allocated and to which queue. For that we can use an SSD like we did for the original metadata.
More information [here]().

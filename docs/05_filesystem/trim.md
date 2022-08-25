# Efficient trim algorithm

Concurrent, parallel, state machine.

Each erase zone has an array of 1 bit per per sector/word (state).
(bloof filter with perfect hash)

512 mb every 16 TB of SSD.

## Delete ops

YFS marks the bit for each sector involved as 1.

If sum of 1s greater than threeshold, invoke compact ops.

## Compact ops

- New, empty zone is selected and frozen.
- YFS freeze the state, and send it to compact process. 
- Any further delete is stored in a temp queue
- Compact process read state, and issue batch copy command for each valid sector to yfs
- When done, YFS issue erase zone command
- YFS apply pending deletes ops


# Advantages

- Very fast
- Never blocking
- Ultra low latencies

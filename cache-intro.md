# cpu-perf-docs
**Read Previous:[Problems Derived from Concepts](./problems-derived.md)**
## Cacheing
Today we are issuing problem whose source is not the processor, but external hardware; which is the RAM.

### The Problem - RAM
The RAM is considered fast hardware, when comparing it to hard disks, and even SSDs. In comparison to CPU, RAM is **VERY** slow. When reading memory from the RAM the operation's latency costs many cycles; The CPU is stalled because it needs the value to work on.

### The Solution - Cache
The Cache is memory hardware which is embedded in the CPU hardware. It usually made from fast SRAM memory hardware (like registers), instead of cheaper DRAM hardware (like main memory). While the CPU requests memory (`lw` instruction for example), it tries to get it from the cache. Only if the memory does not exist in the cache, the CPU requests the data from the RAM; While the CPU fetches the requested address from the RAM, it fetches big block of memory, as it is likely that the next memory address to be requested is close to the current address.

### How it works
#### Level 1
When CPU needs memory, it checks if the address already exist in the closest cache, which is **Level 1 Cache**. This level is the fastest, it takes around 2-3 cycles and has capacity of hundreds of KB.
Level 1 cache is not only exclusive to the processor, but to the core itself.\
When memory is found in the cache it is called **Cache Hit**, if not it is called **Cache Miss**, and the higher the Hit:Miss rate, the better.
#### Level 2
This level is the fall back when the memory isn't exist in level 1. It is bigger, cheaper and slower than level 1, but still much faster than accessing main memory. It usually exists per core, costs around 10 cycles (more or less), and has capacity of few MB.
#### Level 3
This is the slowest, and biggest cache level. It is shared by all the cores of the processor (board with 2 processors or more, has L3 caches as the number of processors). L3 costs around tens-hundreds of cycles, which is still faster than main memory; It has capacity of 100 MB more or less.

#### What Else
Some other facts about cache memory you should know:
* It contains both data and instructions, they are usually stored in different segments
* The is separate to cache lines, which are the units of which memory is stored in cache. For example, cache line in 64-bit ARM processor would be 64 bytes.
* As for developers (and also compilers), writing fragmented instructions (code) or data may harm performance significantly.

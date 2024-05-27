# Performance Driven Concepts

As mentioned in the previous chapter, the industry needed to improve processor's speed, even though they barely can increase the amount of operations in second. There are some techniques used to improve performance\
**note**: not all of them relates to the clock cycles barrier, each technique's motive is explained

## Data Locality

Processor almost never works without inputs, we always provide instructions to run, and data to run on. The data is retrieved from (and then stored to) memory slot, and accessing them takes much more times than the cycles themselves, which stalls the CPU. The solution to the problem is to create in-chip memory cache, it holds blocks of data in with faster access speed, and as long as the processor does not require accessing data not-yet-fetched, the CPU works on his faster copy.

## Multi Cores

Probably the most known technique. If we can't improve our core's performance, why not adding more? This is what happens today in every processor. Each processor have N cores, and different cores can perform independent operations in parallel, so as long as operations are not interfered with each other, the processor's throughput is doubled.

## SMT

SMT (Simultaneous MultiThreading) is, using the same core to process 2 different threads. When operations cause CPU stalling, like I/O access or floating point operations (which are slower the integer operations), the physical core can run another task on the currently unused resources.

## Pipeline Overlapping

Instructions in the CPU are pipelined, each instruction have 5 phases:
* IF = instruction fetching
* ID = instruction decoding
* EX = executing
* MEM = memory accessing
* WB = writeback

For each instruction those steps happens one by one, but for multiple instructions the operations happen in parallel.

IF [N operation] --> ID [(N-1) operation];
ID [(N-1) operation] --> EX [(N-2) operation];
EX [(N-2) operation] --> MEM [(N-3) operation];
MEM [(N-3) operation] --> WB [(N-4 operation)];

When the chip's implementation is divided by steps, throughput can be increased up to 5 times the original throughput.

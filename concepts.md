# Performance Driven Concepts

As mentioned in the previous chapter, the industry needed to improve processor's speed, even though they barely can increase the amount of operations in second. There are some techniques used to improve performance\
**note**: not all of them relates to the clock cycles barrier, each technique's motive is explained

## Multi Cores

Probably the most known technique. If we can't improve our core's performance, why not adding more? This is what happens today in every processor. Each processor have N cores, and different cores can perform independent operations in parallel, so as long as operations are not interfered with each other, the processor's throughput is doubled.

## Data Locality

Processor almost never works without inputs, we always provide instructions to run, and data to run on. The data is retrieved from (and then stored to) memory slot, and accessing them takes much more times than the cycles themselves, which stalls the CPU. The solution to the problem is to create in-chip memory cache, it holds blocks of data in with faster access speed, and as long as the processor does not require accessing data not-yet-fetched, the CPU works on his faster copy.
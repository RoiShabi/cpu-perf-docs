# cpu-perf-docs
**Read Previous:[Caching (intro)](./cache-intro.md)**
## Cache Coherence
Cache is a solution for a problem at any core count, but when mixing multicore system, some complications emerge.\
Let's examine the following code to illustrate the issue with cache as presented earlier:
``` c++
bool is_running = false;

# thread_A
int seconds = 0;
while(is_running){ // keep running until signal received from thread_B
    this_thread::sleep(seconds(1)); // sleep for second
    ++seconds;
    printf("%d seconds passed from the beginning\n", seconds);
}

# thread_B
string input;
do{
    getline(cin, input); // read user input
}while (input=="exit") // keep looping until "exit" input
is_running = false; // signal thread_A to stop
```
\* In our example we assume `printf` and `getline` don't cause context-switch and OS code. This is for keeping the example simple and informative

We examine what suppose to happen, the starting point for our simulation:
* from thread_A perspective:
  * `is_running` is in cache, value is false
  * `seconds` is in cache, value is 8 (arbitrary number)
* from thread_B perspective:
  * user keeps inserting inputs as long as "exit" is not provided

<ins>Flow of events in our simulated situation:</ind>
1. thread_B is receiving the "exit" input
2. thread_B is writing false to cache memory which is mapped to `is_running` address
3. thread_A is loading the value of is_running, from **cache** to register
4. thread_B reads the value, which is **still false** 
5. thread_B is iterating the loop again
6. thread_B is loading **again** the value of `is_running`, from **cache** to register
7. redo steps 4-6

As seen in the simulation above, while thread_B changes the value of `is_running`, thread_A is not aware of the change. The reason thread_A is not aware is that it loads its memory from the local cache, which is not synchronized with the main memory, let alone thread_B's cache.

### What can we do?
We can use protocols which organize all accesses to shared memories. Those protocols are hardware implemented, and the protocol we use is not something we can choose (unless you buy different computer). That said, the fact we don't need to understand them, doesn't mean we shouldn't. So for the rest of the article, we are facing the protocol issue from the perspective of a microcontrollers engineer whose job is to choose the best cache protocol to his new microcontroller.

We have 3 questions to answer before deciding what protocol to use.

#### Directory vs Snooping
* QUESTION: **WHICH** being do I communicate with when I want to access data? (either read or write)
* ANSWER:
  * Directory - There is one being which keeps record of each cache processors cache status, and confirms any new read/write request.
  * Snooping - Processors communicate with each other to inform of any cache read/write.

#### Write-Back vs Write-Through
* QUESTION: **When** do I communicate with cache management being?
* ANSWER:
  * Write-Back - Unlike read operations, write is done directly to main memory
  * Write-Through - Data is written to cache, another events cause synchronizing cache with main memory

#### Write-Invalidate vs Write-Update
* QUESTION: **WHAT** happens to other readers when write operation happens?
* ANSWER:
  * Write-Invalidate - Readers of updated memory are notified their cached data is no longer valid, means whenever they want to access these address they need to reload from main memory
  * Write-Update - Readers are updated their cached memory is not updated, and they receive the new updated value so they can change it without accessing main memory.

### External Sources
Any combination of answers can lead us to different protocol. There are some known protocols like: Berkeley protocol, Synapse protocol, etc.
Those protocols are classified by the memory states they support: **M**odified, **E**xclusive, **O**wner, **S**hared, **I**nvalid. The classes are:
* SI - Shared, Invalid
* MSI - Modified, Shared, Invalid
* MEI - Modified, Exclusive, Invalid
* MES - Modified, Exclusive, Shared
* MESI - Modified, Exclusive, Shared, Invalid
* MOSI - Modified, Owned, Shared, Invalid
* MEOSI - Modified, Exclusive, Owned, Shared, Invalid

How they work and which one is better in what scenario is out of this scope (at least for now), but it is recommended to read more about them.
Some external materials:
* https://people.eecs.berkeley.edu/~pattrsn/252F96/Lecture18.pdf
* https://en.wikipedia.org/wiki/Cache_coherency_protocols_(examples)

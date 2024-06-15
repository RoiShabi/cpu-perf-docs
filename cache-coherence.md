# cpu-perf-docs
**Read Previous:[Caching (intro)](./cache-intro.md)**
## Cache Coherence
Cache is a solution for a problem at any core count, but when mixing multicore system, some complications emerge.\
Let's examine the following code to illustrate the issue with cache as presented earlier:
``` c++
bool is_running = false;

# thread_A
int seconds = 0;
while(is_running){
    this_thread::sleep(seconds(1));
    ++seconds;
    printf("%d seconds passed from the beginning\n", seconds);
}

# thread_B
string input;
do{
    getline(cin, input);
}while (input=="exit")
is_running = false;
```
\* In our example we assume `printf` and `getline` don't cause context-switch and OS code. This is for keeping the example simple and informative
We examine what suppose to happen:
* from thread_A perspective:
  * is_running is in cache, value is false
  * seconds is in cache, value is 8 (arbitrary number)
* from thread_B perspective:
  * user keeps inserting inputs as long as "exit" is not provided

In our simulated situation:
1. thread_B is receiving now the "exit" input
2. thread_B is writing false to cache memory which is mapped to `is_running` address
3. thread_A is loading the value of is_running, from **cache** to register
4. thread_B reads the value, which is **still false** 
5. thread_B is iterating the loop again
6. thread_B is loading **again** the value of `is_running`, from **cache** to register
7. redo steps 4-6

As seen in the simulation above, while thread_B changes the value of `is_running`, thread_A is not aware of the change. The reason thread_A is not aware is that it loads its memory from the local cache, which is not synchronized with the main memory, let alone thread_B's cache.
# cpu-perf-docs
**Read Previous: [Atomics](./atomics.md)**

## Compare Exchange operations
In the c/cpp standard libraries we have variations of compare_exchange operations, the main 2 are `compare_exchange_weak` and `compare_exchange_strong` (fun fact, rust doesn't have equivalent method to `compare_exchange_weak`). They work mostly thw same: according to the compilation architecture, it is translated to either LL/SC  implementation, or CAS (with some setup in both cases).

Both compare_exchange operations returns the result status the operation (succeeded or not). The major difference between them is, that `compare_exchange_weak` may face false-negative results, commonly referred as spurious fails. This means, the `compare_exchange_weak` version can return false even if it could perform the operation. Wy would that be a feature?

### When Architecture Really Matter
As said in previous chapter, LL/SC and CAS implement the same ideal, differently. The issue is, that on LL/SC, is that it is not really single instruction. The usage of multiple instruction creates a gap in which bad stuff can happen. The store-conditional operation may fall even if the address's value was not changed at all. Events like this occur when context switch happens, but the may also happen if address on the same cache line is being touched, since LL/SC checks if the **cache-line** is still in-tact (so spurious fail may also happen on changes on cache state).

On CISC architectures (at least Intel's x86) those scenarios don't happen, so there is not much of a performance boost. On RISC systems the usage of `compare_exchange_strong` requires some extra steps to ensure a failure happens from the right reasons.

### Where To Use Each 

- `compare_exchange_weak` - commonly used on lock-free algorithms of data structures, where a thread loops over compare_exchange until the operation is succeeded, on those case the thread usually would not encounter a failure, and it can benefit from the reduced overhead. In a case the operation failed, the thread can do the operation again.
- `compare_exchange_strong` - less common, it is used in cases where looping over the compare_exchange operation is not necessary or when the loop overhead is big enough so it is better to avoid looping for even with the price of the fixed overhead

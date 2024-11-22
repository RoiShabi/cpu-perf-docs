# cpu-perf-docs
**Read Previous: [Memory Models](./mem-model.md)**
## Atomics
Atomic Operations, also known as 'atomics', are operations which were born for the purpose of threads' communication. Atomcis provide decisive (find better word) memory accesses in order to keep the code's functionallity the same as the programmer intended. In order to do so, atomics harness 3 mechanisms: cache coherency, memory ordering directives, atomic instructions. In order to understand how does it work, we first need to understand the missing part we have not talked about yet - atomic instructions.

### Atomic Instructions
#### Atomic Instructions - Why?
We are already familliar with some CPU instructions, like load & store, add, xor. A race condition might occur when using them to communicate between threads. In the following 2 examples we are going to see real world scenarios where threads may corrupt each other by racing on the same resource.

##### Example 1
```C++
int global_counter = 0;
const int limit = 50;

void print_until_limit()
{
    while(global_counter < limit)
    {
        ++global_counter;
        printf("Counter was increased by 1\n");
    }
}

int main()
{
    jthread t1(print_until_limit);
    jthread t2(print_until_limit);
    t1.join();
    t2.join();
}
```

In the example shown here, the expected results are, 2 threads are printing the string for total amount of `limit' lines. This code results unexpected behaviour. That is, because of two reasons:
1. The access of `global_counter` in the the while loop -> The expectation of `global_counter` to be the same from the loop to the incrementation line 
2. The line `++global_counter` -> It is translated to 3 instruction - load, add, store

The issue is, since both threads accessing `global_counter` resource without synchronization, one can depend on value which is not updated. Lets simulate a scenario:
|Timepoint|Thread1 Instruction|Thread1 t0(register)|Thread2 Instruction|Thread2 t0(register)|global_counter|
|--|--|--|--|--|--|
|0|ld t0 `global_counter`|UNDEFINED (uinitialized)|UNDEFINED(not loaded yet)|UNDEFINED(uinitialized)|0|
|1|addi t0 t0 1|0|ld t0 `global_counter`|0|0|
|2|sw t0 `global_counter`|1|addi t0 t0 1|0|0|
|3|UNDEFINED (between iterations)|1|sw t0 `global_counter`|1|1|
|4|UNDEFINED (between iterations)|1|UNDEFINED (between iterations)|1|1|

As can be seen, even after 2 threads increased `global_counter` by 1, it's value is not 2, but only 1.

##### Example 2
```C++
std::string generate_random_str();
void hardware_custom_print(std::string* strings_array, int count);

constexpr size_t SIZE = 100;
std::string print_buffer[SIZE];
bool is_buffer_locked = false;
size_t next_free_index = 0;

// TODO: 2 threads write and one read and print, need for atomic lock-flag
void insertion_work()
{
    while(true)
    {
        std::string new_str = generate_random_str();
        while(is_buffer_locked) ; // Spinlock until no one uses the buffer
        is_buffer_locked = true; // Before proceeding, acquire the lock

        print_buffer[next_free_index] = new_str;
        ++next_free_index;

        is_buffer_locked = false; // Release the lock
    }
}

void print_work()
{
    while(true)
    {
        while(is_buffer_locked) ; // Spinlock until no one uses the buffer
        is_buffer_locked = true; // Before proceeding, acquire the lock
        
        // create local copy for releasing the lock as soon as possible
        std::string local_array_copy[SIZE];
        memcpy(local_array_copy, 0, print_buffer, 0, sizeof(std::string)*next_free_index)
        int local_array_length = next_free_index;

        is_buffer_locked = false; // Release the lock

        hardware_custom_print(local_array_copy, local_array_length);
    }
}

int main()
{
    jthread writer1(insertion_work);
    jthread writer2(insertion_work);
    jthread reader(printer);
    writer1.join();
    writer2.join();
    reader.join();
}
```
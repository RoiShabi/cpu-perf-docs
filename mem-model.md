# cpu-perf-docs
**Read Previous: [Memory Reordering](./mem-reorder.md)**
## Memory Model
### The Need for memory model
In previous chapter we talked about reordering, and why it can be useful. The problem with reordering, is that it can damage the functionality.\
Here is an example:
```C
int find_first(int* arr, int size, int expected)
{
    int i = 0;
    while(i < size)
    {
        if (arr[i] == expected)
        {
            return i;
        }
        i++;
    }

    return -1;
}
```

Now imagine moving the `if` block above while declaration, then the code looks like that:
```C
int find_first(int* arr, int size, int expected)
{
    int i = 0;
    if (arr[i] == expected)
    {
        return i;
    }
    while(i < size)
    {
       i++;
    }

    return -1;
}
```
Now the method is completely broken. In order to prevent cases like that, we need set of rules to determine which optimization are allowed, and which are not. **Memory Model** is the union of those rules.

### What Models Exists
The 3 memory models we are going to talk about are:
1. Sequential Consistency
2. Acquire-Release Ordering
3. Relaxed Ordering

Usualy people start explaining from Sequential Consistency, but we will do the opposite so it will be more clear.

### Relaxed Ordering
Operations are executed at fully flexible order. This is what actually happen to your code when you dont use synchronization directives (usually atomics, we will discuss them later). It means that the order of execution may vary to the extend of the current thread's observation, in other words, as long as from single-theraded prespective the functionality is OK.
```c++
struct point{int x; int y;};

struct triangle{
    point first;
    point second;
    point third;
};


triangle global_triangle;
bool is_ready = false;

void GenerateObjectThread()
{
    global_triangle.frist = point(1,2);
    global_triangle.second = point(3,2);
    global_triaangle.third = point(2, 3);

    is_ready = true;
}

void PrintTriangleThread()
{
    while (is_ready == false);

    DrowTriangle(global_triangle);
}

int main()
{
    jthread t1(PrintTriangleThread);
    jthread t2(GenerateObjectThread);
    t1.join();
    t2.join();
}
```
In the folloing example, the intuitive result is, `PrintTriangleThread()` waits until `is_ready` is true, and then prints the triangle which is already filled. It reality, this may not be the result. The order of execution can alter in a way that `PrintTriangleThread()` prints the Triangle before checking the flag, or `GenerateObjectThread()` sets the flag before writing data to the `global_triangle`. As long as one of the alterations occur, the result would be invalid object printed to the screen.

### Sequential Consistency
Sequential consistency means, code executes operations in the same order they appear in the code. This may sounds obvious, but after learning about Relaxed ordering, we can understand why it is needed. Imagine we have sequential consistent compiler, and very primitive CPU, the code from previuos section, would never drow an invalid triangle. Of course no modern processor / compiler is strict enough for assumptions like that to be acceptable, so programming languages provides us mechanisms to specify for some operations to be sequential consistent (we will explore them on later chapters).

### Acquire Release Ordering
Acquire Release model helps us achiving more determined code then Relaxed ordering, while still allowing the processor/compiler to have more freedom then Sequential Consistency. Acquire-Relaese appears in pairs, and they say that:
* Acquire - All memory operations after the Acquire cannot be ordered before it, while operations above the Acquire can be ordered below it.
* Release - All memory operations before the Release cannot be ordered after it, while operations below the Release can be ordered above it.

*In this examples, we use imaginary macros named ORDER_ACQUIRE, ORDER_RELEASE, they are used to help visualizing the scenario.
```c++
triangle global_triangle;
bool is_ready = false;
bool is_thread1_running = false;
bool is_thread2_running = false;

void GenerateObjectThread()
{
    is_thread1_running = true;

    global_triangle.frist = point(1,2);
    global_triangle.second = point(3,2);
    global_triaangle.third = point(2, 3);
    ORDER_RELEASE(is_ready) = true;

    is_thread1_running = false;
}

void PrintTriangleThread()
{
    is_thread2_running = true;

    while (ORDER_ACQUIRE(is_ready) == false);
    DrowTriangle(global_triangle);

    is_thread2_running = false;
}

int main()
{
    jthread t1(PrintTriangleThread);
    jthread t2(GenerateObjectThread);
    t1.join();
    t2.join();
}
```

Brief over the changes from the Relaxed Order example:
1. `is_thread1_running` and `is_thread2_running` globals were added.
2. `GenerateObjectThread()` stores `is_ready` with `ORDER_RELEASE`
3. `PrintTriangleThread()` reads `is_ready` with `ORDER_ACQUIRE`

The most important thing to mention: the current example **always produce valid results**. While this code won't drow invalid triangle, it may cause some undesired results for one trying to check the thread's status. To explain, we are adding one more thread (for simplification, it is entirly sequential consistent):

```c++
void ObserveFlagsThread()
{
    point default_point(0,0);
    while(global_triangle.first == default_point); //we exit the loop only after thread 1 started working
    bool copy_thread1_running = is_thread1_running;
    if(is_ready == false) //indicates the tread still working
        assert(copy_thread1_running);
}
```

According to basic logic, if the thread passes the `while`, and `is_ready` has not been set yet, the value of `copy_thread1_running` should always be true. When dealing with Acquire-Release, it is not always true, and the code might fail the assertion. The reason it may fail is, that even if the ORDER_RELEASE forced `is_thread1_running` to be set to true before, `is_ready` is set to true, the line `is_thread1_running = false` at the end of the method, can accur before the ORDER_RELEASE happen. Worth mentioning, the opposite can happen on ORDER_ACQUIRE, meaning `is_thread2_running = true` can happen after the loop of thread 2 and not before.

### Sum Up
All those methods, may be relevant in your programming, as the synchronization method depends on what do you try to achive. Those models come in use when working with atomics, which are the next topic.

**Read Next: [Atomic Operations](./atomics.md)**
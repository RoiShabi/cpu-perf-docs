# cpu-perf-docs
**Read Previous:[Memory Reordering](./mem-reorder.md)**
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

*In the examples, we use imaginary macros named ORDER_RELAXED, ORDER_ACQUIRE, ORDER_RELEASE, ORDER_SEQ_CST, they are used to help visualizing the scenarios. 
### Relaxed Ordering
Operations exists in an undetermine order of execution. This is what actually happen to your code when you dont use synchronization directives (usually atomics, we will discuss them later). It means that the order of execution may vary to the extend of the current thread's observation.
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
    thread(PrintTriangleThread);
    thread(GenerateObjectThread);
}
```
In the folloing example, the intuitive result is, `PrintTriangleThread()` waits until `is_ready` is true, and then prints the triangle which is already filled. It reality, this may not be the result. The order of execution can alter in a way that `PrintTriangleThread()` prints the Triangle before checking the flag, or `GenerateObjectThread()` sets the flag before writing data to the `global_triangle`. As long as one of the alterations occur, the result would be invalid object printed to the screen.

### Acquire Release Ordering

### Sequential Consistency
Sequential consistency means, code executes operations in the same order they appear in the code.
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
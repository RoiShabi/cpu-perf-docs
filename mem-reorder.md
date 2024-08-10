# cpu-perf-docs
**Read Previous:[Performance Driven Concepts](./concepts.md)**
## What is Memory Reordering

Programs use data all the time. In fact, most of program's instructions are either data access (read/write), or data manipulation (calculations/bitwise ops). \When compiling, your code isn't translated to assembly. Another code, with equivalent results, is translated to assembly. As Herb Sutter (has amazing lectures online) said, in the name of your compiler/processor/cache:
> "No, it's much better to **execute a different program**. Hey, don't complain. It's for your own good. You really wouldn't want to execute that *dreck* you actually wrote."


### What does the compiler do?
Compiler optimizes performance by many techniques, among them is changing the flow of the data or the instructions. Let's give some examples.

#### Loop Jamming
In the next code, we have 2 arrays, which are supposed to be filled following the next rule: each element's value is it's index.
```C
void fill_2_arrays(int *arr1, int *arr2, int size_of_arrs)
{
    for(int i = 0; i < size_of_arrs; i++)
    {
        arr1[i] = i;
    }
    for(int i = 0; i < size_of_arrs; i++)
    {
        arr2[i] = i;
    }
}
```

In this code, 90% of the instructions (compare, i++, jump to loop beginning), are not the actual operation. Most of the compilers can merge the loops to 1 loop:
```C
void fill_2_arrays(int *arr1, int *arr2, int size_of_arrs)
{
    for(int i = 0; i < size_of_arrs; i++)
    {
        arr1[i] = i;
        arr2[i] = i;
    }
}
```

#### Data Localization
```C
void fill_zero(int *arr, int size, int *external_index)
{
    for(; *external_index < size; external_index++)
    {
        arr[external_index] = 0;
    }
}
```
This code is very expansive, because we request accessing `external_index` a lot. \
What can the compiler do? It can replace `external_index` with CPU register, like that
```C
void fill_zero(int *arr, int size, int *external_index)
{
    //assembly start
    push t0; //dump current value of temp register
    lw t0 0(external_index); //load external_index to the temp register
    //assembly end

    for(; *t0 < size; t0++)
    {
        arr[t0] = 0;
    }

    //assembly start
    sw t0 0(external_index);
    pop t0;
    //assembly end

}
```
Now, the new code does not update the memory after each `t0` operation, but only at the end of the method.

**Read Next:[Memory Model](./mem-model.md.md)**
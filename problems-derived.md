# cpu-perf-docs
**Read Previous:[Performance Driven Concepts](./concepts.md)**
## Problems Derived from Concepts

In previous chapter we talked about techniques used to improve performance. They really improve performance significantly, but they also introduce new problems. Let's talk about them.

### Race Conditioning

In multicore system, some cores might access the same data. Let's see an example.
* Given dual-core processor, with core_A and core_B.
* The code in the examples are RISC-V based, but they show a cross-architecture, logical issue.
* Memory address 0x00 is a counter which starts at 0, and the cores increase the counter's value by 1 each time
* The code the cores run in a loop is:
  - lw t0, 0(zero)
  - addi t0, t0, 1
  - sw t0, 0(zero)

##### Desired Flow
The flow we expect at first glance is:
```
core_A -> lw t0, 0(zero) # t0 = 0\
core_A -> addi t0, t0, 1 # t0 = 1\
core_A -> sw t0, 0(zero) # **address 0x00** = 1

core_B -> lw t0, 0(zero) # t0 = 1\
core_B -> addi t0, t0, 1 # t0 = 2\
core_B -> sw t0, 0(zero) # **address 0x00** = 2

core_A -> lw t0, 0(zero) # t0 = 2\
core_A -> addi t0, t0, 1 # t0 = 3\
core_A -> sw t0, 0(zero) # **address 0x00** = 3

core_B -> lw t0, 0(zero) # t0 = 3\
core_B -> addi t0, t0, 1 # t0 = 4\
core_B -> sw t0, 0(zero) # **address 0x00** = 4
```

#### In Practice
As we know, the cores run in parallel, so the dont wait one for another to finish, so what can happen is:
```
core_A -> lw t0, 0(zero) # t0 = 0\
core_B -> lw t0, 0(zero) # t0 = 0

core_A -> addi t0, t0, 1 # t0 = 1\
core_B -> addi t0, t0, 1 # t0 = 1

core_A -> sw t0, 0(zero) # **address 0x00** = 1\
core_B -> sw t0, 0(zero) # **address 0x00** = 1
```
\* Other things can happen to. It is not unpredictable by definition, but real life system are too complex. Real time processors and systems exist to solve similar issues, but it is beyond scope.

### Branch Stalling

### Pipeline Suboptimal Utilization
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

### Pipeline Stalling
In pipelined system, sequential instructions processed in different stages. It allows one to reduce execution time. Let's see the example discussed earlier.
```
lw t0, 0(zero)
addi t0, t0, 1
sw t0, 0(zero)
```
Here when `lw` instruction is executed, `addi` is decoded and `sw` is fetched. Now let's check one more example:
```
# This code is an infinite loop, in which t0 is incremented, until it reaches a trashhold (t1), then t0 is set to 0

# Legend
# t0: loop counter
# t1: loop limit
# t2: branch A counter
# t3: branch B counter

I  . beq t0, t1, 16     # if (t0 == t1) go to line 4
II . addi t0, t0, 1     # BRANCH A: t0++
III. addi t2, t2, 1     # BRANCH A: t2++
IV . jal zero, 12       # BRANCH A: skip BRANCH B
V  . addi t0, zero, 0   # BRANCH B: reset counter
VI . addi t3, t3, 1     # BRANCH B: t3++
VII. jal zero, -24      # go to loop's start
```
Now we shall simulate the code with t0=0 and t1=2:
|Cycle No.|Instruction Fetch|Instruction Decode|Instruction Execute|Memory Access|Write Back|Comments|
|---|---|---|---|---|---|---|
|1|(I) beq t0, t1, 16|NO WORK|NO WORK|NO WORK|NO WORK||
|2|(II) addi t0, t0, 1|(I) beq t0, t1, 16|NO WORK|NO WORK|NO WORK||
|3|(III) addi t2, t2, 1|(II) addi t0, t0, 1|(I) beq t0, t1, 16|NO WORK|NO WORK||
|4|(IV) jal zero, 12|(III) addi t2, t2, 1|(II) addi t0, t0, 1|(I) beq t0, t1, 16|NO WORK||
|5|(V) addi t0, zero, 0|(IV) jal zero, 12|(III) addi t2, t2, 1|(II) addi t0, t0, 1|(I) beq t0, t1, 16||


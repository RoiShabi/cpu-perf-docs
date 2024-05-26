# cpu-perf-docs

## The Story of CPU 

If there is one thing we use as much as oxygen these days, is CPU. It is all around us: our computer, our phone, smart TVs, even our cars. In general, it's a chip which does simple math things and communicate with other electronic devices. CPU is not the only hardware capable of those things, but it is programmable, means you can create logics to run on the CPU without the need of production lines. Since everyone has CPU in their PC or phone, One can create his logic and make it run almost everywhere.

## CPU Main Components

In general, CPU contains 4 components: Control Unit, ALU (Arithmetic Logic Unit), Registers, I/O interfaces.

### Control Unit

The control unit is the circuitry which translates machine code to electrical signals for the inner components, it also wires the ALU results to I/O interfaces and vice versa.

### ALU

ALU is the unit doing the bitwise math operations. It adds, subtracts, XORs, NOTs etc.

### I/O Interfaces

Those are units within the processor, which interact with main memory, other processors, hard disks, monitors and so on.

### Registers

Register file is memory a memory storage of data which is only relevant in the current context of the CPU, and they mostly hold data between following operations. Some registers hold the address of the current instruction, the address the top of the stack, and more.

## When Performance began to matter

CPUs today do billions of operations per seconds!! It might sound a lot, but for today's workloads it is not much. Even Windows OS running Word while playing music on YouTube is probably laggy.\
In the past there was a simple solution: wait. Until recent years, each year or two the semiconductor industry produced faster processors. They reduced the size of transistor, therefore doubling the amount of cycles (correlates to operations per second) the chip can withstand.

In recent years these phenomena started breaking. The size of the transistors today is mere hundreds of atoms. On that scale quantum uncertainties become to corrupt the validity of the results.

Today our processors are much more complex than what was explained before, in order to achieve the demands of the market.
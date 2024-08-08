# cpu-perf-docs
**Read Previous:[Performance Driven Concepts](./concepts.md)**
## What is Memory Reordering

Programs use data all the time. In fact, most of program's instructions are either data access (read/write), or data manipulation (calculations/bitwise ops). When compiling code, your code isn't translated to assembly, but another code, with equivalent results, is translated to assembly. As Herb Sutter (has amazing lectures online) said, in the name of your compiler/processor/cache:
> "No, it's much better to **execute a different program**. Hey, don't complain. It's for your own good. You really wouldn't want to execute that *dreck* you actually wrote."

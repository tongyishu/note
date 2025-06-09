ARMv8有3个内存屏障指令：DMB（Data Memory Barrier）、DSB（Data Synchronization Barrier）、ISB（Instruction Synchronization Barrier）。

严格程度：DMB < DSB < ISB。

这3个指令的内存屏障原理在这里不展开叙述（涉及CPU Cache相关知识，我也没有深入研究），现在只给出这3个指令的功能。

# DMB

保证所有在它前面的**访存操作（load/store）** 都执行完毕后，才执行在它后面的**访存操作（load/store）** 。

```c
asm volatile("dmb ld" ::: "memory"); // read

asm volatile("dmb st" ::: "memory"); // write
```

# DSB

保证所有在它前面的**访存操作（load/store）** 都执行完毕后，才执行在它后面的**指令操作（all）** 。

```c
asm volatile("dsb ld" ::: "memory"); // read

asm volatile("dsb st" ::: "memory"); // write
```

# ISB

保证所有在它前面的**指令操作（all）** 都执行完毕后，才执行在它后面的**指令操作（all）** 。

```c
asm volatile("isb" : : : "memory");
```

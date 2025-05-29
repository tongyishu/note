```c
#include <stdio.h>

int main(int argc, char *argv[])
{
	int a = 10, b = 10, c;

	__asm__("movl %1, %%eax; addl %2, %%eax; movl %%eax, %0;"
		: "=r"(c)
		: "r"(a), "r"(b)
		: "%eax");

	printf("c = %d\n", c);
	return 0;
}
```

以上程序实现了 c = a + b 的运算，将输出 c = 20。

AT&T汇编指令中，寄存器以%%开头，操作数以%开头，常数以$开头，1Byte/2Byte/4Byte/8Byte分别对应后缀b/w/l/q。
例子中的%0代表c，%1代表a，%2代表b，以操作数出现的先后顺序作为索引。

C语言内联的汇编语句以asm或__asm__开头，二者是等价的。扩展汇编的模板如下：

```c
__asm__("asm code" : "output" : "input" : "cloberred list");
```

***asm code*** 是AT&T语法下的汇编代码。如例子中的movl，addl语句。

***output*** 是输出操作数，多个以逗号隔开。例子中的"r"为寄存器约束，表示变量c可以存入任何一个通用寄存器。"="为约束修饰符，被修饰的变量是只写的。更多的约束可以参见gcc的官方文档。

***input*** 是输入操作数，约束同output。

***cloberred list*** 是内容被修改的设备列表。如果指令修改了eax通用寄存器，则需要在list里面添加"%eax"; 如果指令修改了条件码寄存器，则需要在list里面添加"cc"; 如果指令修改了内存，则需要在list里面添加"memory"。

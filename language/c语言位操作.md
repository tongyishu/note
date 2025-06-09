定义32位寄存器变量`reg`时，往往会遇到这种情况：一个变量只使用了unsigned int的部分bits，其它bits留作其它用处。比如：

```bash
 31                              15              7 6 5 4 3 2 1 0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           reserved            |               |port |         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

port变量只占用了3个bit（5/6/7），对于port变量的操作只能使用移位运算。那么该如何操作？

由于port只使用了3bit， 0 <= port <= 7， 这里以port = 5为例：

1、定义一个port的掩码mask = 0x000000E0

2、使用掩码对unsigned int的指定bits赋值：

```c
reg &= ~mask;

reg |= (port) << __builtin_ctz(mask);
```

3、使用掩码对unsigned int的指定bits取值：

```c
port = (reg & mask >> __builtin_ctz(mask))
```

上述操作中，mask = 0x000000E0二进制展开后，第5/6/7bit为1，其它bit为0。`__built_ctz(x)`返回x的二进制数最低位起0的个数，直到遇到1为止，`__built_ctz`为gcc的内置函数。

可以将上述操作封装为宏：SET_BITS和GET_BITS，方便操作，具体代码实现如下：

```c
#include <stdio.h>

#define SET_BITS(reg, msk, val)                                                \
	do {                                                                   \
		(reg) &= ~(msk);                                               \
		(reg) |= ((val) << __builtin_ctz(msk));                        \
	} while (0)
#define GET_BITS(reg, msk) (((reg) & (msk)) >> __builtin_ctz(msk))

int main(int argc, char *argv[])
{
	unsigned int reg = 0;
	unsigned int msk = 0xE0;

	SET_BITS(reg, msk, 3);
	printf("reg = %08X\n", reg);
	printf("val = %u\n\n", GET_BITS(reg, msk));

	SET_BITS(reg, msk, 5);
	printf("reg = %08X\n", reg);
	printf("val = %u\n\n", GET_BITS(reg, msk));

	SET_BITS(reg, msk, 7);
	printf("reg = %08X\n", reg);
	printf("val = %u\n\n", GET_BITS(reg, msk));
	return 0;
}
```

程序输出如下：

```bash
# gcc -o out out.c && ./out
reg = 00000060
val = 3

reg = 000000A0
val = 5

reg = 000000E0
val = 7
```

其中`0x60 = 0b01100000`，`0xA0 = 0b10100000`，`0xE0 = 0b11100000`。

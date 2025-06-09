零长度数组是gnu/gcc的实用性扩展，又称为可变数组。使用时在结构体的最后位置声明一个长度为0的数组，这样这个结构体就成为了一个可变长的结构体。对于编译器而言，这个长度为0的数组成员仅仅是一个符号，不会占用任何空间。在结构体中它只代表一个偏移量，为一个不可修改的地址常量。

```c
// 零长度数组
struct test {
	int num;
	int dat[0];
};

// 数组指针
struct test {
	int num;
	int *dat;
};
```

零长度数组相比于数组指针，优势在于：

- 节省了数组指针占用的空间;
- 一次性申请/释放内存（结构体+数组），不用分两次申请和释放（申请时先申请结构体内存，再申请数组内存：释放时先释放数组内存，再释放结构体内存）。

# 零长度数组的使用实例

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct test {
	int num;
	int dat[0];
};

struct test *test_alloc(int num)
{
	struct test *tst;

	/*
		      dat
		      |
		      v
	+-------------+-----+-----+-----+-----+-----+-----+-----+-----+
	| struct test | int | int | int | int | int | int | int | ... |
	+-------------+-----+-----+-----+-----+-----+-----+-----+-----+
		      | ------------------> num <-------------------- |
	*/
	tst = (struct test *)malloc(sizeof(struct test) + sizeof(int) * num);
	if (!tst)
		return NULL;
	tst->num = num;
	memset(tst->dat, 0, sizeof(int) * num);
	return tst;
}

void *test_free(struct test *tst)
{
	free(tst);
}

int main(int argc, char *argv[])
{
	struct test *tst;

	tst = test_alloc(10);
	if (!tst)
		return -1;

	printf("num: %u\n", tst->num);
	printf("tst: 0x%08x\n", tst);
	printf("dat: 0x%08x\n", tst->dat);
	printf("sizeof(struct test): 0x%0u\n", sizeof(struct test));
	test_free(tst);
	return 0;
}

```

上述程序运行的结果为：

```bash
# gcc -o test test.c && ./test
num: 10
tst: 0xd91d2410
dat: 0xd91d2414
sizeof(struct test): 0x4
```

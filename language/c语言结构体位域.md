# 位域的定义

```c
struct ss {
    unsigned int a : 5;
    unsigned int b : 7;
    unsigned int c : 11;
    unsigned int d : 9;
};
```

如上，struct ss结构体使用了位域，一个unsigned int的类型（32bit）被划分为a、b、c、d四个区域，其中a : 5bit，b : 7bit，c : 11bit，d : 9bit。总共加起来为32bits，刚好为一个unsigned int大小。

当然，也可以使用char，short int，int，long long这些基本类型作为位域的载体。

# 位域的操作

需要注意的是：使用结构体声明的位域，是可以直接使用 "." 运算符操作的。如下程序：

```c
#include <stdio.h>

struct ss {
	unsigned int a : 5;
	unsigned int b : 7;
	unsigned int c : 11;
	unsigned int d : 9;
};

int main(int argc, char *argv[])
{
	struct ss s;

	s.a = 5;
	s.b = 7;
	s.c = 11;
	s.d = 9;

	printf("s.a = %u\n", s.a);
	printf("s.b = %u\n", s.b);
	printf("s.c = %u\n", s.c);
	printf("s.d = %u\n", s.d);
	return 0;
}
```

可以直接使用类似s.a = 5这样的操作进行赋值，也可以直接使用 unsigned int a = s.a这样的操作进行取值。以上程序的支行结果如下：

```bash
# gcc -o out out.c && ./out
s.a = 5
s.b = 7
s.c = 11
s.d = 9
```

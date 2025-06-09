# 什么是\_\_attribute\_\_((constructor))？

`__attribute__`机制是gnu c的特性，用来设置函数属性，变量属性和类型属性。

`__attribute__`前后都有两个下划线，并且后面会紧跟**两对**小括号，括号里面是相应的`__attribute__`参数。

语法格式：`__attribute__((XXX))`。

# 不带优先级的\_\_attribute\_\_((constructor))？

`__attribute__((constructor))`和`__attribute__((destructor))`用来设置函数属性。

`__attribute__((constructor))`修饰的函数在main主函数之前执行。

`__attribute__((destructor))`修饰的函数在main主函数之后执行。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

__attribute__((constructor)) static void before_main()
{
	printf("run before main.\n");
}

__attribute__((destructor)) static void after_main()
{
	printf("run after main.\n");
}

int main(int argc, char *argv[])
{
	printf("run main.\n");
	return 0;
}

```

运行结果如下：

```bash
# gcc -o test test.c && ./test
run before main.
run main.
run after main.
```

# 带优先级的\_\_attribute\_\_((constructor))？

`__attribute__((constructor))`还可以带优先级，优先级影响函数执行的先后顺序。

`__attribute__((constructor(101)))`，优先级为101，数字越小，优先级越高，**越先**被执行。

`__attribute__((destructor(102)))`，优先级为102，数字越小，优先级越高，**越后**被执行。

自定义的优先级需要从101开始，因为0~100优先级被用于内部实现。

如果使用0~100的优先级，编译时会告警：`constructor priorities from 0 to 100 are reserved for the implementation`。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

__attribute__((constructor(101))) static void before_main1()
{
	printf("run before main1.\n");
}

__attribute__((destructor(101))) static void after_main1()
{
	printf("run after main1.\n");
}

__attribute__((constructor(102))) static void before_main2()
{
	printf("run before main2.\n");
}

__attribute__((destructor(102))) static void after_main2()
{
	printf("run after main2.\n");
}

int main(int argc, char *argv[])
{
	printf("run main.\n");
	return 0;
}
```

运行结果如下：

```bash
# gcc -o test test.c && ./test
run before main1.
run before main2.
run main.
run after main2.
run after main1.
```

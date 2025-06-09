这是我在调试代码时遇到的一个问题，首先给出这个坑的简化代码：

```c
#include <stdio.h>

int main()
{
        unsigned char a = 0;
        unsigned char b = 254;
        unsigned char c = a - b;
        unsigned char d = (a - b) >> 1;
        unsigned char e = c >> 1;
        printf("c=%u\n", c);
        printf("d=%u\n", d);
        printf("e=%u\n", e);
        return 0;
}
```

我以为的输出：

```bash
c=2
d=1
e=1
```

实际上的输出：

```bash
c=2
d=129
e=1
```

解释一下原因：计算d时，a和b都是放在寄存器里面的，32位机器上一般是4字节，即一个`unsigned int`整型。计算时将（a - b）的值强转为`unsigned int`类型了。因此，在计算完（a - b）之后，必须先将其结果强转为`unsigned char`类型，再做移位。正确的做法如下（见i的计算和输出）：

```c
#include <stdio.h>
int main()
{
        unsigned char a = 0;
        unsigned char b = 254;
        unsigned char c = a - b;
        unsigned char d = (a - b) >> 1;
        unsigned char e = c >> 1;
        unsigned int  f = a - b;
        unsigned int  g = f >> 1;
        unsigned int  h = (unsigned char)g;
        unsigned int  i = ((unsigned char)(a - b)) >> 1;
        printf("c=%u\n", c);
        printf("d=%u\n", d);
        printf("e=%u\n", e);
        printf("f=0x%08x\n", f);
        printf("g=0x%08x\n", g);
        printf("h=%u\n", h);
        printf("i=%u\n", i);
        return 0;
}
```

以上代码的输出如下：

```bash
c=2
d=129
e=1
f=0xffffff02
g=0x7fffff81
h=129
i=1
```

其中h的值和d的值一样，这也是d的值为什么等于129的原因！！！

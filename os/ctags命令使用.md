# ctags支持的语言

ctags最先是用来生成C代码的tags文件，后来扩展成可种类语言的tags。可以使用`ctags --list-maps`查看支持的语言及对应的文件后缀。也可以使用`ctags --list-maps=C`指定语言查看：

```bash
# ctags --list-maps=C
C        *.c
```

# ctags支持的数据类型

ctags支持给struct或class等类型打tag，可以通过`ctags --list-kinds`查看，生成的tags文件可以直接使用vim打开，kind列即为tags的类型。也可使用`ctags --list-kinds=C`指定语言查看：

```bash
# ctags --list-kinds=C
d  macro definitions
e  enumerators (values inside an enumeration)
f  function definitions
g  enumeration names
h  included header files
l  local variables [off]
m  struct, and union members
p  function prototypes [off]
s  structure names
t  typedefs
u  union names
v  variable definitions
x  external and forward variable declarations [off]
z  function parameters inside function or prototype definitions [off]
L  goto labels [off]
D  parameters inside macro definitions [off]
```

# ctags给指定语言打tag

可以使用--languages=C为C语言相关的文件打tag，同时可以使用--langmap=C:+.c.h指定文件的后缀，比如：`ctags --languages=C --langmap=C:+.c.h --list-maps=C`

```bash
# ctags --languages=C --langmap=C:+.c.h --list-maps=C
C        *.c *.h
```

# ctags指定生成的tag文件路径

如果不指定-f或-o选项，ctags默认在当前目录下生成名为"tags"的文件，可以通过-f或-o指定生成的tag文件，比如：`ctags -R -f mytags .`

```bash
# ctags -R -f mytags .
# ls
mytags
```

# ctags从标准输入中读取文件名

ctags的-L选项可以从文件中读取要打tag的文件名，如果指定"-L -"则是从标准输入中读取文件名，比如：`find -name *.c | ctags -L -`

这个与xargs ctags -a有类似的功能，其中-a是 --append=yes，比如：`find -name *.c | xargs ctags -a`

```bash
# find -name *.c | ctags -L -
# ls
tags
```

# ctags排除指定的文件和文件夹

ctags的--exclude选项可以排除指定的文件，加上-R选项可以排除指定的文件夹，比如：`ctags --exclude=./m4 -R .`

```bash
# ctags --exclude=./m4 -R .
# ls
tags
```

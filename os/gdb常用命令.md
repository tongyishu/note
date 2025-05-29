# gdb 常用命令


| **命令**                         | **功能**                                                                                                                                                                                                                                          |
| :--------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| gdb --verison                    | 查看gdb版本信息                                                                                                                                                                                                                                   |
| set pagination off               | 当gdb输出信息过多时，gdb会暂停输出，并打印`---Type to continue, or q to quit---`的提示信息，set pagination off会取消提示信息的打印                                                                                                                |
| set args                         | 设置被调试程序的启动参数，比如：`set args --no-script`（假定被调试程序有`--no-script`启动参数）                                                                                                                                                   |
| show args                        | 查看被调试程序的启动参数                                                                                                                                                                                                                          |
| set step-mode on                 | gdb在默认情况下不会进入没有调试信息的函数，比如库函数printf，该命令可以设置不跳过不带调试信息的函数，此时可以使用调试汇编程序的方法调试函数                                                                                                       |
| set prompt                       | 设置命令提示符，默认为"(gdb) "，比如：`set prompt mygdb`，提示符就会变成"mygdb"                                                                                                                                                                   |
| finish                           | 退出函数，函数会执行完，并打印返回值                                                                                                                                                                                                              |
| return                           | 退出函数，函数不会继续执行，而是直接返回，可以指定函数的返回值为expr                                                                                                                                                                              |
| file                             | 加载要调试程序文件，比如：`file /usr/bin/w`                                                                                                                                                                                                       |
| info proc                        | 查看程序的启动参数                                                                                                                                                                                                                                |
| info variables                   | 查看全局变量和静态变量                                                                                                                                                                                                                            |
| info args                        | 查看栈帧的参数                                                                                                                                                                                                                                    |
| info functions                   | 查看ELF文件的所有函数                                                                                                                                                                                                                             |
| info frame                       | 查看栈帧信息                                                                                                                                                                                                                                      |
| info break                       | 查看断点信息                                                                                                                                                                                                                                      |
| step                             | 单步陷入执行                                                                                                                                                                                                                                      |
| next                             | 单步执行                                                                                                                                                                                                                                          |
| call                             | 调用函数                                                                                                                                                                                                                                          |
| frame                            | 选择栈帧                                                                                                                                                                                                                                          |
| break *                          | 当调试汇编程序，或没有调试信息的程序时，可以在程序地址上设置断点，比如：`break *0x04000522`在程序入口入设置断点，可以通过`info files`查看程序的Entry point                                                                                        |
| break N                          | 在当前文件的第line行设置断点，比如：`break 7`                                                                                                                                                                                                     |
| break :                          | 在file文件的line行设置断点，比如：`break main.c:23`                                                                                                                                                                                               |
| break ... if                     | 设置条件断点，只有当condifion满足中才命名断点，比如：`break 10 if i==101`，只有当i == 101时才触发断点                                                                                                                                             |
| ignore                           | 忽略num号断点count次，断点号可以通过`info break`查看，比如：`ignore 1 10`，忽略1号断点10次，只有在第11次命中时才触发1号断点                                                                                                                       |
| tbreak                           | 设置临时断点，用法与break一致，断点命中一次后删除该断点                                                                                                                                                                                           |
| enable                           | 使能断点                                                                                                                                                                                                                                          |
| disable                          | 禁用断点                                                                                                                                                                                                                                          |
| watch                            | 设置观察点，当一个变量的值发生变化时，程序会暂停，比如：`watch a`，当a的值发生变化时程序会停止执行; `watch a thread 3`，当3号线程改变a的值时，程序会暂停，其它线程改变时，程序不会暂停                                                            |
| rwatch                           | 设置只读观察点，当发生读取变量的值时，程序会暂停                                                                                                                                                                                                  |
| awatch                           | 设置读写观察点，当发生读写变量的值时，程序会暂停                                                                                                                                                                                                  |
| info watchpoints                 | 查看所有观察点                                                                                                                                                                                                                                    |
| set can-use-hw-watchpoints <0/1> | 设置是否启用硬件观察点，0禁用，1启用，默认优先使用硬件观察点，不支持硬件观察点时才使用软件观察点硬件观察点：程序执行过程中，使用硬件的寄存器协助检测变量的值，执行速率快软件观察点：单步执行，每执行一步检测变量是否发生变化                      |
| catch                            | 设置事件捕捉点，常用于捕捉系统调用catch syscall ，比如：`catch syscall read`，当有read系统调用时，程序会暂停; `catch syscall 1`，当有1号系统调用时，程序会暂停系统调用清单一般位置<br/>/usr/share/gdb/syscalls/amd64-linux.xml文件中              |
| set logging on                   | 打开gdb日志功能，记录gdb的执行过程，方便后续分析，默认的日志文件为gdb.txt                                                                                                                                                                         |
| set logging file                 | 将gdb的日志文件设置为filepath                                                                                                                                                                                                                     |
| print *@                         | 打印ptr数组中num个元素，比如：`p *a@10`，打印数组a的10个连续的元素                                                                                                                                                                                |
| set print array-indexes on       | 打印数组元素的同时，打印元素的下标索引                                                                                                                                                                                                            |
| info locals                      | 查看当前函数的局部变量                                                                                                                                                                                                                            |
| backtrace                        | 查看调用栈                                                                                                                                                                                                                                        |
| backtrace full                   | 查看调用栈的同时，打印各个调用栈的局部变量                                                                                                                                                                                                        |
| info proc mappings               | 查看进程内存的映射信息                                                                                                                                                                                                                            |
| info files                       | 查看ELF文件的具体信息                                                                                                                                                                                                                             |
| p ''::                           | 查看file文件中的variable变量，比如：`p 'test.c'::g_test`，查看test.c文件中的g_test静态变量                                                                                                                                                        |
| ptype                            | 打印变量或结构体，比如：ptype a，查看a变量的类型`ptype /o struct test`，查看struct test结构体及其内存分布                                                                                                                                         |
| ctrl+x, a                        | 进入图形化界面调试（源码模式），也可以在gdb启动时设--tui参数，再按一次ctrl+x, a，会退出图形化界面（源码模式）                                                                                                                                     |
| ctrl+x, o                        | 在图形化界面下，焦点转移到下方的指令窗口（按上下键翻历史命令），而不是在上方的源码窗口（按上下键滚动源码），再按一次ctl+x, o会转移焦点上方的源码窗口                                                                                              |
| gdb attach                       | 调试正在运行的进程，比如：gdb attach 1234，调试进程号为1234的进程                                                                                                                                                                                 |
| gdb                              | gdb加载ELF文件和其coredump文件，加载后会自动进入到coredump点                                                                                                                                                                                      |
| .gdbinit                         | gdb启动时会读取和加载HOME的当前目录下的.gdbinit文件，可以在.gdbinit文件中对gdb进行启动配置，也可以自定义函数，比如：`set print pretty on`，打印格式优化; `set confirm off`，退出时不打印提示信息;                                                 |
| set follow-fork-mode             | parent调试父进程，子进程不受影响，child调试子进程，父进程不受影响                                                                                                                                                                                 |
| set detach-on-fork off           | 同时调试父子进程，gdb默认只会追踪父进程，子进程会独立运行，并且调试其中一个进程时，另一个进程挂起                                                                                                                                                 |
| info inferiors                   | 查看被调试的进程的状态                                                                                                                                                                                                                            |
| inferior                         | 切换到num号进程调试                                                                                                                                                                                                                               |
| info threads                     | 查看所有进程，前面带"*"号的为当前线程                                                                                                                                                                                                             |
| thread                           | 切换到num号线程                                                                                                                                                                                                                                   |
| thread apply all backtrace       | 查看所有线程的堆栈                                                                                                                                                                                                                                |
| set scheduler-locking on         | 只调试当前线程，其它线程不运行                                                                                                                                                                                                                    |
| info sharedlibrary               | 查看程加载的共享链接库，比如：`info sharedlibrary`，查看已加载的所有共享链接库`info sharedlibrary libm*`，正则匹配以libm开头的共享链接库                                                                                                          |
| set history save on              | 保留gdb调试历史，存放在当前目录下的`.gdb_history`文件中，gdb默认是不保留的                                                                                                                                                                        |
| set history filename             | 设置gdb调试历史的保留文件的路径，默认为当前目录下的.gdb_history文件                                                                                                                                                                               |
| pwd                              | 查看当前gdb的工作目录                                                                                                                                                                                                                             |
| cd                               | 切换gdb的工作目录                                                                                                                                                                                                                                 |
| set env =                        | 设置被调试程序的环境变量，比如：`set env LD_PRELOAD=/lib/x86_64-linux-gnu/libpthread.so`                                                                                                                                                          |
| info signals                     | 查看信号屏蔽情况                                                                                                                                                                                                                                  |
| signal                           | 向被调试程序发送信号，比如：`signal SIGHUP`                                                                                                                                                                                                       |
| set variable =                   | 设置变量的值，比如：`set variable i=8`，设置局部变量的i=8; `set variable main::ptr = "for test"`，设置main函数中的ptr指针指向"for test"字符串; `set variable *(int*)0x123456 = 42`，将地址0x123456强转为int*指针，并赋值42                        |
| set write on                     | 使用gdb修改ELF文件的内容，gdb默认是以只读的形式加载ELF文件的，该命令可指定为ELF文件可写，比如：`set write on`; `file ./test`; `set*(int*)0x123456=15`; 退出gdb后，ELF文件中0x123456偏移处的变量的值被修改为15，也可以通过`gdb -write`的形式来指定 |

# gdb x命令

GDB 中的 `x` 命令（全称为 `examine`）用于**检查内存内容** ，支持多种格式和单位查看内存数据。以下是详细用法和示例：

## 基本语法

```bash
x/[数量][格式][单位] <地址或表达式>
```

* **数量** ：要显示的元素个数（默认 1）。
* **格式** ：数据的显示方式（如十六进制、字符、指令等）。
* **单位** ：每个元素的大小（如字节、字、双字等）。
* **地址** ：内存地址（可直接写变量、寄存器或数值）。

## 常用格式说明符


| 格式符 | 含义         | 示例               |
| -------- | -------------- | -------------------- |
| `x`    | 十六进制     | `x/x 0xffff0000`   |
| `d`    | 十进制       | `x/d &my_var`      |
| `u`    | 无符号十进制 | `x/u $rsp`         |
| `o`    | 八进制       | `x/o 0x404000`     |
| `t`    | 二进制       | `x/t main`         |
| `a`    | 地址         | `x/a ptr`          |
| `c`    | 字符         | `x/c buffer`       |
| `s`    | 字符串       | `x/s 0x5555556000` |
| `i`    | 汇编指令     | `x/i $pc`          |

## 常用单位说明符


| 单位符 | 大小          | 示例                   |
| -------- | --------------- | ------------------------ |
| `b`    | 字节（1字节） | `x/4xb 0x7fffffffecc0` |
| `h`    | 半字（2字节） | `x/2hx $sp`            |
| `w`    | 字（4字节）   | `x/8wx main`           |
| `g`    | 大字（8字节） | `x/2gx 0x401000`       |

GDB 中的 `handle` 命令用于**控制调试器对信号（Signals）的处理方式** ，可以指定当程序收到特定信号时，GDB 是否停止程序、打印信息或传递信号给程序。以下是详细用法和示例：

# gdb handle命令

## **基本语法**

```bash
handle [信号] [动作]
```

* **信号** ：信号名称（如 `SIGSEGV`）或编号（如 `11`）。
* **动作** ：多个动作参数组合，控制 GDB 对信号的处理方式。

## **动作参数说明**


| 动作      | 作用                                                               |
| ----------- | -------------------------------------------------------------------- |
| `nostop`  | 收到信号时**不停止程序** （默认某些信号如 `SIGALRM` 会继续运行）。 |
| `stop`    | 收到信号时**暂停程序** （默认大部分信号如 `SIGSEGV` 会停止）。     |
| `noprint` | 收到信号时**不打印提示信息** 。                                    |
| `print`   | 收到信号时**打印提示信息** （默认启用）。                          |
| `pass`    | 将信号**传递给程序处理** （程序可以捕获或忽略信号）。              |
| `nopass`  | **不传递信号给程序** ，由 GDB 直接处理（程序无法捕获该信号）。     |
| `ignore`  | 等同于`nopass` + `nostop` + `noprint`（完全忽略信号）。            |

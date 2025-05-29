tail 命令打印每个文件的最后10行到标准输出。如果有多个FILE，会在每个文件的打印内容前面加上该文件的文件名。 如果没有文件，或者当文件为"-"时，从标准输入读取内容。

tail 命令一般在调试程序或定位问题时，用于跟踪日志文件。常用的命令如下：

# tail -f /var/log/messages

跟踪/var/log/messages的日志内容。

tail -f等同于–follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止。

# tail -n 3 /var/log/messages

输出/var/log/messages日志的最后三行。

tail -n K输出文件的最后K行，而不是默认的10行。

# tail -F /var/log/messages

等同于–follow=name --retry，根据文件名进行追踪，并保持重试，即该文件被删除或改名后，如果再次创建相同的文件名，会继续追踪。

# tail -f /var/log/messages --pid 10345

跟踪/var/log/messages的日志输出，当10345号进程退出时，停止追踪。

tail --pid PID 用于追踪指定进程（PID），当进程退出或异常终止时，停止追踪文件内容。

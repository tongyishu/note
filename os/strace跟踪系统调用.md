strace监控用户空间进程和内核的交互，比如系统调用、信号传递、进程状态变更等。

```bash
# strace -tt -f touch xxx.txt
15:08:59.092376 execve("/usr/bin/touch", ["touch", "xxx.txt"], 0x7ffdedd7d948 /* 24 vars */) = 0
15:08:59.093399 brk(NULL)               = 0x560f9dcae000
15:08:59.094156 arch_prctl(0x3001 /* ARCH_??? */, 0x7ffc786bbcc0) = -1 EINVAL (Invalid argument)
15:08:59.094740 mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f38b1285000
15:08:59.095276 access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
15:08:59.095731 openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
15:08:59.096145 newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=24791, ...}, AT_EMPTY_PATH) = 0
15:08:59.099186 mmap(NULL, 24791, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f38b127e000
15:08:59.099724 close(3)                = 0
```

```bash
# strace -tt -f -e trace=open -p 22121
Process 22121 attached with 13 threads
[pid 22145] 17:57:45.558768 open("/proc/21856/status", O_RDONLY) = 2174
[pid 22121] 17:57:46.056771 open("/sys/class/infiniband/mlx5_0/ports/1/hw_counters/out_of_buffer", O_RDONLY) = 2174
[pid 22145] 17:57:46.558786 open("/proc/21856/status", O_RDONLY) = 2174
```

-f 跟踪由fork调用所产生的子进程

-tt 在输出中的每一行前加上时间信息,微秒级

-p 指定进程号

-T 显示每个系统调用所消耗的时间

-x 以十六进制形式输出非标准字符串

-e 控制如何跟踪

-e trace=file 只跟踪文件操作的系统调用

-e trace=process 只跟踪进程控制的系统调用

-e trace=network 只跟踪网络相关的系统调用

-e trace=signal 只跟踪信号相关的系统调用

-e trace=open,close,read,write 只跟踪指定的系统调用（这里为open,close,read,write）

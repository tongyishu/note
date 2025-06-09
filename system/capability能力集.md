# 什么是capability？

capability能力集是对root权限的细化，实现按需授权，从而减小系统的安全攻击面。

capability是针对进程而言的，而非用户。当前linux系统已知有以下38种能力集：

* CAP_AUDIT_CONTROL 启用和禁用内核审计；改变审计过滤规则；检索审计状态和过滤规则
* CAP_AUDIT_READ 允许读取审计日志
* CAP_AUDIT_WRITE 允许将记录写入内核审计日志
* CAP_BLOCK_SUSPEND 使用可以阻止系统挂起的特性
* CAP_CHOWN 允许修改文件所有者的权限
* CAP_DAC_OVERRIDE 忽略文件的DAC访问限制
* CAP_DAC_READ_SEARCH 忽略文件读及目录搜索的DAC访问限制
* CAP_FOWNER 忽略文件属主ID必须和进程用户ID相匹配的限制
* CAP_FSETID 允许设置文件的setuid位
* CAP_IPC_LOCK 允许锁定共享内存片段
* CAP_IPC_OWNER 忽略IPC所有权检查
* CAP_KILL 允许对不属于自己的进程发送信号
* CAP_LEASE 允许修改文件锁的FL_LEASE标志
* CAP_LINUX_IMMUTABLE 允许修改文件的IMMUTABLE和APPEND属性标志
* CAP_MAC_ADMIN 允许MAC配置或状态更改
* CAP_MAC_OVERRIDE 覆盖MAC(Mandatory Access Control)
* CAP_MKNOD 允许使用mknod()系统调用
* CAP_NET_ADMIN 允许执行网络管理任务
* CAP_NET_BIND_SERVICE 允许绑定到小于 1024 的端口
* CAP_NET_BROADCAST 允许网络广播和多播访问
* CAP_NET_RAW 允许使用原始套接字
* CAP_SETGID 允许改变进程的GID
* CAP_SETFCAP 允许为文件设置任意的capabilities
* CAP_SETPCAP 允许向其他进程转移能力以及删除其它进程的能力
* CAP_SETUID 允许改变进程的UID
* CAP_SYS_ADMIN 允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等
* CAP_SYS_BOOT 允许重新启动系统
* CAP_SYS_CHROOT 允许使用chroot()系统调用
* CAP_SYS_MODULE 允许插入和删除内核模块
* CAP_SYS_NICE 允许提升优先级及设置其他进程的优先级
* CAP_SYS_PACCT 允许执行进程的BSD式审计
* CAP_SYS_PTRACE 允许跟踪任何进程
* CAP_SYS_RAWIO 允许直接访问/devport、/dev/mem、/dev/kmem及原始块设备
* CAP_SYS_RESOURCE 忽略资源限制
* CAP_SYS_TIME 允许改变系统时钟
* CAP_SYS_TTY_CONFIG 允许配置TTY设备
* CAP_SYSLOG 允许使用syslog()系统调用
* CAP_WAKE_ALARM 允许触发唤醒系统

# 如何设置和查看程序和进程的capability？

`setcap CAP_DAC_OVERRIDE=eip /usr/bin/vim` 将CAP_DAC_OVERRIDE能力以cap_effective(e)，cap_inheritable(i)，cap_permitted(p)三种位图的方式授权给程序文件/usr/bin/vim。

`getcap /usr/bin/vim` 查看程序文件/usr/bin/vim的能力集。

`getpcaps 10086` 查看pid=10086进程的能力集。

`cat /proc/PID/status`查看进程的状态，能力集信息也在其中。

# capability能力在程序与进程之间的传递

进程和程序文件的能力集有以下三种授权方式（cap_effective、cap_inheritable、cap_permitted）。

`cap_effective`：进程当前实际拥有的能力集合

`cap_inheritable`：进程可以传递给子进程的能力集合

`cap_permitted`：进程被允许拥有的所有能力集合

当程序文件被加载到内存运行时，相应的进程便有了程序文件的能力。

# 一、核心机制与函数原型

`flock` 是 Unix/Linux 系统中用于实现文件锁的系统调用，主要用于进程间同步（IPC），确保多个进程对同一文件的访问不会冲突。它通过为文件描述符关联锁，控制其他进程对文件的读写权限。

`flock` 的本质是为文件描述符fd关联一个锁，通过操作文件描述符控制锁的状态。其函数原型如下：

```c
#include <sys/file.h>
int flock(int fd, int operation);
```

# 二、关键参数

`operation` 参数决定锁的类型和行为，支持以下标志（可组合使用）：

| 标志        | 含义                                                                                                 |
| ----------- | ---------------------------------------------------------------------------------------------------- |
| `LOCK_SH` | 共享锁（读锁）：允许多个进程同时加共享锁，但排斥其他进程的排他锁。                                   |
| `LOCK_EX` | 排他锁（写锁）：仅允许一个进程加锁，排斥其他进程的共享锁或排他锁（默认模式）。                       |
| `LOCK_UN` | 解锁：释放当前进程对文件的锁（进程退出时会自动解锁）。                                               |
| `LOCK_NB` | 非阻塞标志（可与 `LOCK_SH`/`LOCK_EX` 组合）：加锁失败时不阻塞，直接返回错误（`EWOULDBLOCK`）。 |

# 三、锁的核心特性

- 锁与进程生命周期绑定：当进程退出（无论正常终止或崩溃），系统会自动释放该进程持有的所有flock锁，避免死锁。
- 锁与文件描述符fd的关系：若通过dup()或fork()复制文件描述符，新描述符与原描述符共享同一把锁（关闭其中一个描述符不会释放锁）。所有关联的文件描述符关闭后，锁才会被释放（例如：父进程fork子进程，父子进程共享锁，需都退出才释放）。
- 锁的非阻塞模式通过LOCK_NB标志可避免进程无限等待。例如：当进程尝试加排他锁但锁已被占用时，flock会立即返回-1，并设置errno=EWOULDBLOCK（或EAGAIN），进程可选择跳过操作或执行其他逻辑（如重试）。
- 锁的粒度是整个文件，且无法锁目录。flock无法像fcntl的F_SETLK那样锁定文件的部分字节，只能锁定整个文件，细粒度锁需用fcntl。而且flock无法对目录加锁，目录的锁需通过其他方式实现，如fcntl的字节范围锁。
- 锁是全局跨进程的。之所以能实现全局跨进程，在于其锁信息不存在于用户空间或进程内存中，而是由内核文件系统模块直接通过文件的唯一标识（如inode）和进程ID（PID）进行全局管理。内核为每个文件（通过inode标识）维护一个锁列表，记录所有当前持有的flock锁。当进程尝试加锁时，内核会遍历该列表，检查是否存在冲突的锁（如排他锁已被占用），并决定是否允许当前进程加锁。

# 代码示例

flock命令会调用flock()系统调用，这里以flock命令为例:

```bash
# 终端1执行以下命令:
flock -e ./test tail -f /var/log/messages

# 终端2执行以下命令:
flock -e ./test ls ./
```

1. 终端1使用`-e`排他锁，锁住了./test文件，用于执行`tail -f /var/log/messages`命令
2. 终端2使用`-e`排他锁，尝试锁住./test文件，用于执行`ls ./`命令
3. 由于终端1已经锁住了./test文件，导致终端2执行时阻塞
4. 此时`ctrl + c`停止终端1的命令，终端2方得以继续执行

# nng版本

v1.9.0

# nng通信方式

| 通信模式    | 套接字类型                 |
| ----------- | -------------------------- |
| 一对一      | `pair0/pare1`            |
| 请求 - 响应 | `req/rep`、`xreq/xrep` |
| 发布 - 订阅 | `pub/sub`、`xsub`      |
| 管道        | `push/pull`              |
| 总线        | `bus`                    |
| 调查 - 响应 | `survery/respond`        |

以pub/sub模型分析，两个进程的命令如下：

```bash
./nngcat --pub0 --listen "tcp://127.0.0.1:8989" --data "yishu" --interval 3
./nngcat --sub0 --dial "tcp://127.0.0.1:8989" --quoted
```

# nng流程梳理

```c
nni_init
	nni_posix_pollq_sysinit
		nni_posix_pollq_create // 全局事件监听队列，仅1个: nni_posix_global_pollq
			epoll_create1
			nni_posix_pollq_add_eventfd
			nni_posix_poll_thr
	nni_posix_resolv_sysinit
	nni_plat_init
		nni_init_helper
			nni_taskq_sys_init
				nni_taskq_init // 全局任务队列，仅1个: nni_taskq_systq
					nni_thr_init
						nni_taskq_thread // 执行task的线程, 可根据cpu个数配置
			nni_reap_sys_init
			nni_aio_sys_init
				nni_aio_expire_q_alloc // 全局超时队列，最大8个，可根据cpu个数配置: nni_aio_expire_q_list
					nni_thr_init
						nni_aio_expire_loop // 每个超时队列都有1个线程执行该函数
			nni_tls_sys_init
			nni_sp_tran_sys_init // 全局扩展协议链表，仅1个: sp_tran_list
				nni_sp_inproc_register
				nni_sp_ipc_register
				nni_sp_tcp_register
				nni_sp_tls_register
				nni_sp_udp_register
				nni_sp_ws_register
				nni_sp_wss_register
				nni_sp_zt_register
				nni_sp_sfd_register


nng_dial // nng_listener也是类似
	nni_sock_find
	nni_dialer_create
		nni_sock_add_dialer
	nni_dialer_start
		dialer_connect_start
			tcptran_ep_connect // ipc/inproc/ws/tls/zt也是类似
				nng_stream_dialer_dial
					tcp_dialer_dial
						resolv_worker // 交给nng:resolver线程处理
							resolv_task
								getaddrinfo // libc库函数
									connect // 系统调用


nng_recv // nng_send也是类似
	nng_recvmsg
		nni_aio_init
		nni_sock_recv
			sub0_sock_recv // pair/pub/suvery/respond/req/rep也是类似
				sub0_ctx_recv
					nni_aio_begin
					... // 从消息队列中获取消息
					nni_aio_set_msg
					nni_aio_finish // 将aio交给nng:task线程调度处理
		nni_sock_rele
		nni_aio_wait
		nni_aio_result
		nng_aio_get_msg
		nni_aio_fini
```

# nng线程梳理

使用 `./nngcat --sub0 --dial tcp://127.0.0.1:8989 --quoted`命令起了一个进程，`gdb attach`之后线程如下：

```bash
(gdb) i thr
  Id   Target Id                                            Frame 
* 1    Thread 0x7f7931fac740 (LWP 2347102) "nngcat"         0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  2    Thread 0x7f79317aa700 (LWP 2347103) "nng:poll:epoll" 0x00007f79318a80f7 in epoll_wait () from /lib64/libc.so.6
  3    Thread 0x7f7930fa9700 (LWP 2347104) "nng:resolver"   0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  4    Thread 0x7f79307a8700 (LWP 2347105) "nng:resolver"   0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  5    Thread 0x7f792ffa7700 (LWP 2347106) "nng:resolver"   0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  6    Thread 0x7f792f7a6700 (LWP 2347107) "nng:resolver"   0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  7    Thread 0x7f792efa5700 (LWP 2347108) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  8    Thread 0x7f792e7a4700 (LWP 2347109) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  9    Thread 0x7f792dfa3700 (LWP 2347110) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  10   Thread 0x7f792d7a2700 (LWP 2347111) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  11   Thread 0x7f792cfa1700 (LWP 2347112) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  12   Thread 0x7f792c7a0700 (LWP 2347113) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  13   Thread 0x7f792bf9f700 (LWP 2347114) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  14   Thread 0x7f792b79e700 (LWP 2347115) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  15   Thread 0x7f792af9d700 (LWP 2347116) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  16   Thread 0x7f792a79c700 (LWP 2347117) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  17   Thread 0x7f7929f9b700 (LWP 2347118) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  18   Thread 0x7f792979a700 (LWP 2347119) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  19   Thread 0x7f7928f99700 (LWP 2347120) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  20   Thread 0x7f7928798700 (LWP 2347121) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  21   Thread 0x7f7927f97700 (LWP 2347122) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  22   Thread 0x7f7927796700 (LWP 2347123) "nng:task"       0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  23   Thread 0x7f7926f95700 (LWP 2347124) "nng:reap2"      0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  24   Thread 0x7f7926794700 (LWP 2347125) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  25   Thread 0x7f7925f93700 (LWP 2347126) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  26   Thread 0x7f7925792700 (LWP 2347127) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  27   Thread 0x7f7924f91700 (LWP 2347128) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  28   Thread 0x7f7924790700 (LWP 2347129) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  29   Thread 0x7f7923f8f700 (LWP 2347130) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  30   Thread 0x7f792378e700 (LWP 2347131) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
  31   Thread 0x7f7922f8d700 (LWP 2347132) "nng:aio:expire" 0x00007f7931b7e3fc in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
```

这5类线程的名称及运转方式如下：

- nngcat
- nng:poll:epoll // 事件
- nng:resolver   // 事件
- nng:task       // 事件
- nng:aio:expire // 事件 + 计时器
- nng:reap2      // 忙等待

线程间协作如下：

```bash
主线程(nngcat):
│   ├─ 启动事件处理线程(nng:poll:epoll)
│   ├─ 启动DNS解析线程(nng:resolver)
│   ├─ 启动任务处理线程(nng:task)
│   └─ 启动资源回收线程(nng:reap2)
│   └─ 启动aio超时线程(nng:aio:expire)


事件处理线程(nng:poll:epoll): 
│   ├─ 监听FD事件（tcp/ipc等）
│   ├─ 触发tcp_cb/ipc_cb等回调来接收消息
│   ├─ 异步操作：生成aio任务 → aio任务队列 → nng:task线程
│   └─ 待回收资源：放入「资源回收队列」 → nng:reap2线程


DNS解析线程(nng:resolver):
│   ├─ 接收解析请求（来自主线程或事件线程）
│   └─ 异步域名解析，解析完成 → 返回IP地址给nng:poll:epoll线程


任务处理线程(nng:task):
│   ├─ 从aio任务队列获取完成事件
│   ├─ 处理aio回调，执行用户定义的回调函数
│   └─ 接收超时线程提交的超时任务（来自nng:aio:expire）


aio超时线程(nng:aio:expire)
│   ├─ 定时扫描「aio回收队列」，判断aio任务是否超时
│   └─ 超时事件 → 封装为任务 → 提交到nng:task队列


资源回收线程(nng:reap2)
│   ├─ 定时扫描「资源回收队列」，从「资源回收队列」获取待释放对象（如aio、tcp连接）
│   └─ 执行安全释放（内存释放、FD关闭等）
```

# nng相关调用栈

- listen

```bash
#0  0x00007ffff78e8f30 in listen () from /lib64/libc.so.6
#1  0x000055555559f185 in nni_tcp_listener_listen (l=0x5555555d3840, sa=0x5555555d37a8) at /home/kayne/nng/src/platform/posix/posix_tcplisten.c:265
#2  0x000055555559ba18 in tcp_listener_listen (arg=0x5555555d3770) at /home/kayne/nng/src/core/tcp.c:302
#3  0x000055555559a271 in nng_stream_listener_listen (l=0x5555555d3770) at /home/kayne/nng/src/core/stream.c:239
#4  0x000055555558f485 in tcptran_ep_bind (arg=0x5555555d3080) at /home/kayne/nng/src/sp/transport/tcp/tcp.c:1103
#5  0x000055555556a337 in nni_listener_start (l=0x5555555d2840, flags=0) at /home/kayne/nng/src/core/listener.c:410
#6  0x000055555556219a in nng_listener_start (lid=..., flags=0) at /home/kayne/nng/src/nng.c:634
#7  0x0000555555560af4 in main (ac=8, av=0x7fffffffe298) at /home/kayne/nng/src/tools/nngcat/nngcat.c:1159
```

- connect

```bash
#0  0x00007ffff78e8e00 in connect () from /lib64/libc.so.6
#1  0x00007ffff791ee53 in open_socket () from /lib64/libc.so.6
#2  0x00007ffff791f3ae in __nscd_get_mapping () from /lib64/libc.so.6
#3  0x00007ffff791d1cf in __nscd_get_nl_timestamp () from /lib64/libc.so.6
#4  0x00007ffff7904e9b in __check_pf () from /lib64/libc.so.6
#5  0x00007ffff78d260f in getaddrinfo () from /lib64/libc.so.6
#6  0x00005555555865d3 in resolv_task (item=0x5555555d40a0) at /home/kayne/nng/src/platform/posix/posix_resolv_gai.c:178
#7  0x0000555555586a33 in resolv_worker (unused=0x0) at /home/kayne/nng/src/platform/posix/posix_resolv_gai.c:342
#8  0x0000555555573f92 in nni_thr_wrap (arg=0x5555555ce440) at /home/kayne/nng/src/core/thread.c:94
#9  0x000055555557698f in nni_plat_thr_main (arg=0x5555555ce440) at /home/kayne/nng/src/platform/posix/posix_thread.c:266
#10 0x00007ffff7bb817a in start_thread () from /lib64/libpthread.so.0
#11 0x00007ffff78e7dc3 in clone () from /lib64/libc.so.6
```

- sendmsg

```bash
#0  0x00007f9534f95910 in sendmsg () from /lib64/libpthread.so.0
#1  0x000055dfdaadd0f4 in tcp_dowrite (c=0x7f9518000cc0) at /home/kayne/nng/src/platform/posix/posix_tcpconn.c:69
#2  0x000055dfdaadd8b7 in tcp_send (arg=0x7f9518000cc0, aio=0x7f9508000c60) at /home/kayne/nng/src/platform/posix/posix_tcpconn.c:292
#3  0x000055dfdaac9f22 in nng_stream_send (s=0x7f9518000cc0, aio=0x7f9508000c60) at /home/kayne/nng/src/core/stream.c:133
#4  0x000055dfdaabdeb1 in tcptran_pipe_send_start (p=0x7f9508000b60) at /home/kayne/nng/src/sp/transport/tcp/tcp.c:521
#5  0x000055dfdaabdfb9 in tcptran_pipe_send (arg=0x7f9508000b60, aio=0x7f950c000e90) at /home/kayne/nng/src/sp/transport/tcp/tcp.c:545
#6  0x000055dfdaa9d55e in nni_pipe_send (p=0x7f950c000b60, aio=0x7f950c000e90) at /home/kayne/nng/src/core/pipe.c:123
#7  0x000055dfdaaadb60 in pub0_sock_send (arg=0x55dfdb673148, aio=0x7ffebc9a9b90) at /home/kayne/nng/src/sp/protocol/pubsub0/pub.c:247
#8  0x000055dfdaaa0012 in nni_sock_send (sock=0x55dfdb672be0, aio=0x7ffebc9a9b90) at /home/kayne/nng/src/core/socket.c:844
#9  0x000055dfdaa9127e in nng_sendmsg (s=..., msg=0x55dfdb675000, flags=0) at /home/kayne/nng/src/nng.c:198
#10 0x000055dfdaa8f7de in sendloop (sock=...) at /home/kayne/nng/src/tools/nngcat/nngcat.c:590
#11 0x000055dfdaa90c5d in main (ac=8, av=0x7ffebc9aa238) at /home/kayne/nng/src/tools/nngcat/nngcat.c:1196
```

- readv

```bash
#0  0x00007ffff78de650 in readv () from /lib64/libc.so.6
#1  0x00005555555ad371 in tcp_doread (c=0x7fffdc000b60) at /home/kayne/nng/src/platform/posix/posix_tcpconn.c:131
#2  0x00005555555ad70f in tcp_cb (pfd=0x7fffdc000c20, events=1, arg=0x7fffdc000b60) at /home/kayne/nng/src/platform/posix/posix_tcpconn.c:242
#3  0x0000555555578476 in nni_posix_poll_thr (arg=0x5555555cd7c0 <nni_posix_global_pollq>) at /home/kayne/nng/src/platform/posix/posix_pollq_epoll.c:291
#4  0x0000555555573f92 in nni_thr_wrap (arg=0x5555555cd7f8 <nni_posix_global_pollq+56>) at /home/kayne/nng/src/core/thread.c:94
#5  0x000055555557698f in nni_plat_thr_main (arg=0x5555555cd7f8 <nni_posix_global_pollq+56>) at /home/kayne/nng/src/platform/posix/posix_thread.c:266
#6  0x00007ffff7bb817a in start_thread () from /lib64/libpthread.so.0
#7  0x00007ffff78e7dc3 in clone () from /lib64/libc.so.6
```

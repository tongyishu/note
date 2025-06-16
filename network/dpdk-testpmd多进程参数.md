testpmd常用的多进程参数有两个：`--proc-type`和`--file-prefix`，这两个参数都和进程间的共享配置有关。

```bash
root@kayne-VirtualBox:~# ll /var/run/dpdk/vte/
total 1780
drwx------ 2 root root    220 10月 18 21:37 ./
drwx------ 3 root root     60 10月 18 21:36 ./
-rw------- 1 root root  25344 10月 18 21:37 config
srwxr-xr-x 1 root root      0 10月 18 21:37 dpdk_telemetry.v2=
-rw------- 1 root root 397312 10月 18 21:37 fbarray_menseg_2048k-0-0
-rw------- 1 root root 397312 10月 18 21:37 fbarray_menseg_2048k-0-1
-rw------- 1 root root 397312 10月 18 21:37 fbarray_menseg_2048k-0-2
-rw------- 1 root root 397312 10月 18 21:37 fbarray_menseg_2048k-0-3
-rw------- 1 root root 188446 10月 18 21:37 fbarray_menzone
-rw------- 1 root root  12720 10月 18 21:37 hugepage_info
srwxr-xr-x 1 root root      0 10月 18 21:37 mp_socket=
```

**--proc-type** 指定共享配置（hugepage、telemetry、mp_socket）的两个进程的类型，一个是primary（主进程）， 一个是secondary（从进程），primary可以创建共享配置，secondary只能attach共享配置。比如：

```bash
--proc-type=primary   # 启动的dpdk进程为主进程
--proc-type=secondary # 启动的dpdk进程为从进程
```

**--file-prefix** 指定共享配置（hugepage、telemetry、mp_socket）的路径前缀，如果不指定该参数，默认为/var/run/dpdk/rte，该参数通常用于两个独立的primary进程。比如：

```bash
--file-prefix=vhost   # 配置文件位于/var/run/dpdk/vhost目录下
--file-prefix=virtio  # 配置文件位于/var/run/dpdk/virtio目录下
```

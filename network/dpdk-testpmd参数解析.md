dpdk-testpmd 其后的启动参数里面，"--"用于分隔EAL参数和testpmd应用的参数，"--"之前为EAL参数，"--"之后为testpmd APP的参数。

```bash
./dpdk-testpmd -n 4 -l 0,1 -- -i --txd=1024 --rxd=1024
```

参考文档：

https://doc.dpdk.org/dts/test_plans/virtio_smoke_test_plan.html
http://doc.dpdk.org/guides/testpmd_app_ug/intro.html

# EAL参数

-l 指定cpu，如 -l 0,1使用0号和1号cpu，-l 3-6 使用3\~6号cpu。

--main-lcore 设置主线程的core ID。

--socket-mem 为每个NUMA预分配一定数量的内存，如：--socket-mem=1024,2048。

--iova-mode 强制使用指定的IOVA模式。

-b, --block `[domain:]bus:device.func` 忽略指定的PCI设备，如-b 0000:03:00.0，与-a选项互斥。

-a, --allow `[domain:]bus:device.func` 探测指定的PCI设备，如-a 0000:03:00.0，与-b选项互斥。

--no-pci 禁止PCI总线。

--vdev `[,key=val, ...]` 添加一个虚拟设备，如：`--vdev=net_vhost0,iface=vhost-net,queues=1`

--file-prefix 此选项允许在不同的前缀下运行多个独立的DPDK主或从进程，会在`/var/run/dpdk/`目录下生成对应的数据。

# testpmd APP参数

-i, --interactive 交互模式下运行testpmd。

--nb-cores=N 设置转发核的数量为N，N>=1，N默认为1。

--rxq=N 设置RX队列个数为N。

--txq=N 设置TX队列个数为N。

--rxd=N 设置RX队列深度为N。

--txd=N 设置TX队列深度为N。

# testpmd使用实例

**Driver: net_virtio_user && Device: vhost-user**

创建后端，驱动类型为net_vhost（vhost-user），iface=vhost-net指定后端的socket文件，用于前端接管。

```bash
./dpdk-testpmd \
	-n 4 \
	-l 0,1 \
	--no-pci \
	--file-prefix=vhost \
	--vdev=net_vhost0,iface=vhost-net,queues=1 \
	-- \
	-i \
	--nb-cores=1 \
	--txd=1024 \
	--rxd=1024
```

创建前端，驱动类型为net_virtio_user，path=./vhost-net为要接管的后端的socket文件。

```bash
./dpdk-testpmd \
	-n 4 \
	-l 0,1 \
	--no-pci \
	--file-prefix=virtio \
	--vdev=net_virtio_user0,mac=00:01:02:03:04:05,path=./vhost-net,packed_vq=1,mrg_rxbuf=0,in_order=1,vectorized=1,queue_size=1024 \
	-- \
	-i \
	--nb-cores=1 \
	--txd=1024 \
	--rxd=1024
```

**Driver: net_virtio_user && Device: vhost-kernel**

创建后端，驱动类型为net_vhost（kernel），由内核打开path=/dev/vhost-net设备创建一个tap口，暴露给用户面。

创建前端，前端的驱动类型为net_virtio_user，接管内核创建的tap口。

```bash
./dpdk-testpmd \
	-n 4 \
	-l 0,1 \
	--no-pci \
	--vdev=net_virtio_user0,path=/dev/vhost-net,queue_size=1024 \
	-- \
	-i \
	--txd=1024 \
	--rxd=1024
```

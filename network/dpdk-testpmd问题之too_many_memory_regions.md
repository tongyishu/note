# 遇到的问题

在使用以下命令拉起vhost-user device和virtio-user driver时，遇到了"Too many memory regions"问题：

```bash
./dpdk-testpmd \
    -n 4 \
    -l 0,1 \
    --no-pci \
    --file-prefix=vhost \
    --vdev=net_vhost0,iface=./vhost-net-user \
    -- \
    -i \
    --nb-cores=1 \
    --rxq=1 \
    --txq=1 \
    --txd=512 \
    --rxd=512

./dpdk-testpmd \
    -n 4 \
    -l 0,2 \
    --no-pci --file-prefix=virtio \
    --vdev=net_virtio_user0,mac=01:02:03:04:05:06,path=./vhost-net-user,packed_vq=1,mrg_rxbuf=1,in_order=0,vectorized=0 \
    -- \
    -i \
    --nb-cores=1 \
    --rxq=1 \
    --txq=1 \
    --txd=512 \
    --rxd=512
```

遇到的问题现象如下：

```bash
EAL: Detected CPU lcores: 3  
EAL: Detected NUMA nodes: 1  
EAL: Detected static linkage of DPDK  
EAL: Multi-process socket /var/run/dpdk/virtio/mp_socket  
EAL: Selected IOVA mode 'PA'  
TELEMETRY: No legacy callbacks, legacy socket not created  
Interactive-mode selected  
Warning: NUMA should be configured manually by using --port-numa-config and --ring-numa-config parameters along with --numa.  
testpmd: create a new mbuf pool <mb_pool_0>: n=155456, size=2176, socket=0  
testpmd: preferred mempool ops selected: ring_mp_mc  
update_memory_region(): Too many memory regions  
vhost_user_set_memory_table(): Failed to set memory table  
virtio_user_mem_event_cb(): (./vhost-net-user) Failed to update memory table  
update_memory_region(): Too many memory regions  
vhost_user_set_memory_table(): Failed to set memory table  
virtio_user_mem_event_cb(): (./vhost-net-user) Failed to update memory table  

Warning! port-topology-paired and odd forward ports number, the last port will pair with itself.  

Configuring Port 0 (socket 0)  
Port 0: 01:02:03:04:05:06  
Checking link statuses...  
Done  
Error during enabling promiscuous mode for port 0: Operation not supported - ignore  
testpmd> quit  

Stopping port 0...  
Stopping ports...  
Done  
```

# 产生的原因

原因是我使用的是2M的大页内存，memory_regions比较多，vhost侧产生的region有100多个，而virtio侧协商时最多支持VHOST_MEMORY_MAX_NREGIONS（8）个region，超过8个会报错。

```bash
root@kayne-VirtualBox:/dev/hugepages# ls | sort -u
vhostmap_0
vhostmap_1
vhostmap_10
vhostmap_100
vhostmap_101
vhostmap_102
vhostmap_103
vhostmap_104
vhostmap_105
vhostmap_106
...
virtiomap_0
virtiomap_1
virtiomap_2
virtiomap_3
virtiomap_4
virtiomap_5
virtiomap_6
virtiomap_7
```

# 解决的方法

- 一种解决方法是修改源码，将`VHOST_MEMORY_MAX_NREGIONS`修改为1000，亲测有效，需要注意的是要修改两个地方，一个lib/vhost下的，一个drivers/net/virtio下的，然后再编译运行即可。
  dirvers/net/virtio下的：

  ```c
  /* The version of the protocol we support */
  #define VHOST_USER_VERSION    0x1

  #define VHOST_MEMORY_MAX_NREGIONS 8
  struct vhost_memory {
  	uint32_t nregions;
  	uint32_t padding;
  	struct vhost_memory_region regions[VHOST_MEMORY_MAX_NREGIONS];
  };
  ```

  lib/vhost下的：

  ```c
  /* refer to hw/virtio/vhost-user.c */

  #define VHOST_MEMORY_MAX_NREGIONS 8

  #define VHOST_USER_NET_SUPPORTED_FEATURES \
  	(VIRTIO_NET_SUPPORTED_FEATURES | \
  	 (1ULL << VIRTIO_F_RING_PACKED) | \
  	 (1ULL << VIRTIO_NET_F_MTU) | \
  	 (1ULL << VHOST_F_LOG_ALL) | \
  	 (1ULL << VHOST_USER_F_PROTOCOL_FEATURES) | \
  	 (1ULL << VIRTIO_NET_F_CTRL_RX) | \
  	 (1ULL << VIRTIO_NET_F_GUEST_ANNOUNCE))
  ```
- 另外一种解决方法是启动时加上`--single-file-segments`选项，可以将memory文件变为一个：

  ```bash
  ./dpdk-testpmd \
      --single-file-segments \
      -n 4 \
      ...
  ```

此时再查看/dev/hugepages目录，会发现vhost侧和virtio侧的memory map文件都只变成了一个：

```bash
root@kayne-VirtualBox:/dev/hugepages# ls -l
total 757760
-rw------- root root 387973120 11月  2 23:01 vhostmap_0
-rw------- root root 387973120 11月  2 23:01 virtiomap_0
```

# rss reta配置

```bash
# rss reta配置
testpmd> port config <PORT_ID> rss reta (HASH,QUEUE)[,(HASH,QUEUE), ...]

# 查看端口配置的rss_reta
testpmd> show port <PORT_ID> rss reta <SIZE> (MASK0, MASK1, ...)

# hash=0~3分别散列到queue0~3
testpmd> port config 0 rss reta (0,0),(1,1),(2,2),(3,3)

# hash=255散列到queue2, hash=511散列到queue3
testpmd> port config 0 rss reta (255,2),(511,3)

# 打印hash值为0/2对应的queue
testpmd> show port 0 rss reta 64 (5)

# 打印hash值为0/1/2对应的queue
testpmd> show port 0 rss reta 64 (7)

# 打印hash值为0~63对应的queue
testpmd> show port 0 rss reta 64 (-1)

# 打印hash值为0~127对应的queue
testpmd> show port 0 rss reta 128 (-1,-1)

# 打印所有512个条目
testpmd> show port 0 rss reta 512 (-1,-1,-1,-1,-1,-1,-1,-1)

# 打印最后的64条目
testpmd> show port 0 rss reta 512 (0,0,0,0,0,0,0,-1)

# 脚本生成rss reta配置
for ((i = 0; i < 128; i += 8)); do echo -n "port config 0 rss reta "; \
for ((j = i; j < i + 8; j++)); do if test $i -eq $j; then echo -n "($j,1)"; else echo -n ",($j,1)"; fi; \
done; echo; done
```

# rss hash配置

```bash
# rss hash配置
testpmd> port config all rss <RSS_HASH>

# 配置hash_type为ipv4-tcp
testpmd> port config all rss ipv4-tcp

# 关闭端口的rss
testpmd> port config all rss none

# 查看端口配置的rss_hash
testpmd> show port <PORT_ID> rss-hash
```

# rss hash_key配置

```bash
testpmd> port config <PORT_ID> rss-hash-key <RSS_HASH_TYPE> <RSS_HASH_KEY>

# 配置rss_hash_key
testpmd> port config 0 rss-hash-key ipv4-tcp \
6d5a56da255b0ec24167253d43a38f00112233445566778899aabbccddeeff001122334455667788
```

- 代码流程

```c
cmd_config_rss_parsed
	rte_eth_dev_rss_hash_update
		rte_eth_rss_hf_refine
		dev_ops->rss_hash_update

cmd_set_rss_reta_parsed
	rte_eth_dev_rss_reta_update
		dev_ops->reta_update

cmd_config_rss_hash_key_parsed
	port_rss_hash_key_update
		rte_eth_dev_rss_hash_update
			dev_ops->rss_hash_update
```

# promisc配置

- 配置命令

```bash
# 开启/关闭混杂
testpmd> set promisc <PORT_ID> on
testpmd> set promisc <PORT_ID> off
```

- 代码流程

```c
eth_set_promisc_mode
	rte_eth_promiscuous_enable
		dev_ops->promiscuous_enable
	rte_eth_promiscuous_disable
		dev_ops->promiscuous_enable
```

# allmuticast配置

- 配置命令

```bash
# 开启/关闭全组播
testpmd> set allmuti <PORT_ID> on
testpmd> set allmuti <PORT_ID> off
```

- 代码流程

```c
eth_set_allmulticast_mode
	rte_eth_allmulticast_enable
		dev_ops->allmulticast_enable
	rte_eth_allmulticast_disable
		dev_ops->allmulticast_disable
```

# multicast配置

- 配置命令

```bash
testpmd> mcast_addr add    <PORT_ID> <MCAST_ADDRESS>
testpmd> mcast_addr remove <PORT_ID> <MCAST_ADDRESS>
testpmd> mcast_addr flush  <PORT_ID>
testpmd> show port         <PORT_ID> mcast_macs
```

- 代码流程

```c
mcast_addr_add
	eth_port_multicast_addr_list_set
		rte_eth_dev_set_mc_addr_list
			dev_ops->set_mc_addr_list
```

# unicast配置

- 配置命令

```bash
testpmd> mac_addr add    <PORT_ID> <MAC_ADDRESS>
testpmd> mac_addr remove <PORT_ID> <MAC_ADDRESS>
testpmd> mac_addr set    <PORT_ID> <MAC_ADDRESS>
testpmd> set vf mac addr <PORT_ID> <VF_ID> <MAC_ADDRESS>
testpmd> show port       <PORT_ID> macs
```

- 单播列表示意图

```bash
+--------------+
|     mac0     |  index=0, default_mac, `mac_addr set` overwrite;
+--------------+
|     mac1     |  index=1  \
+--------------+              `mac_addr add` append; `mac_addr remove` delete;
|     ...      |  index=n  /
+--------------+
```

- 代码流程

```c
rte_eth_dev_mac_addr_add
	dev_ops->mac_addr_add

rte_eth_dev_mac_addr_remove
	dev_ops->mac_addr_remove

rte_eth_dev_default_mac_addr_set
	dev_ops->mac_addr_set
```

# bond配置

- 代码流程

```c
// example/bond/main.c
main
	rte_eal_init
		rte_bus_scan
		rte_bus_probe
	rte_eth_dev_count_avail
	slave_port_init
		rte_eth_dev_configure
		rte_eth_dev_adjust_nb_rx_tx_desc
		rte_eth_rx_queue_setup
		rte_eth_tx_queue_setup
		rte_eth_dev_start
	bond_port_init
		rte_eth_bond_create
			rte_vdev_init
				insert_vdev
				vdev_probe_all_drivers
					bond_probe // driver->probe, associated with rte_eth_dev and rte_vdev_device
						bond_alloc
							rte_eth_vdev_allocate
								rte_eth_dev_allocate // alloc a unused port_id for "rte_eth_dev"
								rte_zmalloc_socket // alloc memory for "bond_dev_private"
		rte_eth_dev_configure
		rte_eth_dev_adjust_nb_rx_tx_desc
		rte_eth_bond_slave_add
			__eth_bond_slave_add_lock_free
				slave_add // add slave_port_id to "bond_dev_private"
		rte_eth_rx_queue_setup
		rte_eth_tx_queue_setup
		rte_eth_dev_start
		rte_eth_promiscuous_enable
```

# mtu配置

```bash
testpmd> port config mtu <PORT_ID> <MTU>
testpmd> show port info  <PORT_ID>
```

- 代码流程

```c
cmd_config_mtu_parsed
	port_mtu_set
		eth_dev_validate_mtu
		rte_eth_dev_set_mtu
			dev_ops->mtu_set
```

# vlan配置

- 配置vlan的硬件卸载能力（是否拥有该能力）
```bash
testpmd> port config <PORT_ID> rx_offload vlan_strip  on|off
testpmd> port config <PORT_ID> rx_offload vlan_filter on|off
testpmd> port config <PORT_ID> rx_offload vlan_extend on|off
testpmd> port config <PORT_ID> rx_offload qinq_strip  on|off
testpmd> port config <PORT_ID> tx_offload vlan_insert on|off
testpmd> port config <PORT_ID> tx_offload qinq_insert on|off
```

- 使能vlan的硬件卸载能力（是否使用该能力）

```bash
testpmd> vlan set strip      on/off <PORT_ID>
testpmd> vlan set filter     on/off <PORT_ID>
testpmd> vlan set qinq_strip on/off <PORT_ID>
testpmd> vlan set extend     on/off <PORT_ID>
```

- 设置rx的vlan过滤

```bash
testpmd> rx_vlan add/rm all <PORT_ID>
testpmd> rx_vlan add/rm <VLAN_ID> <PORT_ID>
```

- 设置tx的vlan_tpid/vlan_pvid

```bash
# qinq配置内/外层的tpid
testpmd> vlan set inner tpid <TP_ID> <PORT_ID>
testpmd> vlan set outer tpid <TP_ID> <PORT_ID>

# 1、untagged的帧自动添加pvid;
# 2、qinq时配置外层pvid, 内层pvid由用户报文携带
testpmd> tx_vlan set pvid <PORT_ID> <VLAN_ID> on|off
```

- 代码流程

```c
cmd_vlan_offload_parsed
	rx_vlan_strip_set
		rte_eth_dev_set_vlan_offload
			dev_ops->vlan_offload_set
	rx_vlan_qinq_strip_set
		rte_eth_dev_set_vlan_offload
			dev_ops->vlan_offload_set
	rx_vlan_filter_set
		rte_eth_dev_set_vlan_offload
			dev_ops->vlan_offload_set
	vlan_extend_set
		rte_eth_dev_set_vlan_offload
			dev_ops->vlan_offload_set

cmd_rx_vlan_filter_parsed
	rx_vft_set
		rte_eth_dev_vlan_filter
			dev_ops->vlan_filter_set

cmd_rx_vlan_filter_all_parsed
	rx_vlan_all_filter_set
		rx_vft_set
			rte_eth_dev_vlan_filter
				dev_ops->vlan_filter_set

cmd_vlan_tpid_parsed
	vlan_tpid_set
		rte_eth_dev_set_vlan_ether_type
			dev_ops->vlan_tpid_set

cmd_tx_vlan_set_pvid_parsed
	tx_vlan_pvid_set
		rte_eth_dev_set_vlan_pvid
			dev_ops->vlan_pvid_set
```

# dfx诊断

- 查看端口信息

```bash
# 查看端口信息
testpmd> show port info   <PORT_ID>
testpmd> show port stats  <PORT_ID>
testpmd> show port xstats <PORT_ID>
testpmd> clear port stats
```

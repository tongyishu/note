官方文档：
https://doc.dpdk.org/dts/test_plans/checksum_offload_test_plan.html

```bash
The support of RX/TX L3/L4 Checksum offload features by Poll Mode Drivers consists in:

On the RX side:

Verify IPv4 checksum by hardware for received packets.
Verify UDP/TCP/SCTP checksum by hardware for received packets.
On the TX side:

IPv4 checksum insertion by hardware in transmitted packets.
IPv4/UDP checksum insertion by hardware in transmitted packets.
IPv4/TCP checksum insertion by hardware in transmitted packets.
IPv4/SCTP checksum insertion by hardware in transmitted packets (sctp length in 4 bytes).
IPv6/UDP checksum insertion by hardware in transmitted packets.
IPv6/TCP checksum insertion by hardware in transmitted packets.
IPv6/SCTP checksum insertion by hardware in transmitted packets (sctp length in 4 bytes).
```

# 设置csum转发模式

```bash
set fwd csum
```

该动作会将使用struct fwd_engine csum_fwd_engine转发引擎，调用
pkt_burst_checksum_forward函数进行报文转发，pkt_burst_checksum_forward执行以下几个步骤：

```bash
rte_eth_rx_burst # 收包
process_inner_cksums # 处理内层报文的csum
process_outer_cksums # 处理外层报文的csum
rte_eth_tx_burst # 发包
```

# csum硬件卸载

```bash
csum set ip hw 2  # 设置port2的ip csum offload
csum set tcp hw 2 # 设置port2的tcp csum offload
csum set udp hw 2 # 设置port2的udp csum offload
```

该动作会先将rte_port[2]（rte_port与ovs中的port类似，都与rte_eth_devices对应）的dev_conf.txmode.offload字段的csum相关域置1，在
pkt_burst_checksum_forward转发时，再通过dev_conf.txmode.offload配置，将每个报文的rte_mbuf中的ol_flags字段相关域置1，从而带给硬件。

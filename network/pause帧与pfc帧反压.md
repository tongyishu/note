pause帧和pfc帧均为网络流控机制，用于在网络传输过程中将数据流的发送暂停一段时间，以防止数据丢失或者网络性能下降。其中pause的粒度是网卡接口，pfc的粒度是网卡队列。pfc机制在本质上是对pause机制的一种改进和细化。

# pause帧格式

```bash
+--------------------------------+
|     Destination Address        |   // 6B, 目的MAC地址, 取值固定为 01-80-c2-00-00-01
+--------------------------------+
|     Source Address             |   // 6B, 源MAC地址
+--------------------------------+
|     Ethernet Type              |   // 2B, 以太网帧类型, 固定为 0x8808
+--------------------------------+
|     Control Opcode             |   // 2B, 控制码, 取值为 0x0001
+--------------------------------+
|     Pause Time                 |   // 2B, 暂停时间, 即接收方请求发送方停止发送数据帧的时间长度
+--------------------------------+
|     Pad                        |   // 42B, 按以太网最小帧要求进行填充, 全 0 
+--------------------------------+
|     CRC                        |   // 4B, 循环冗余校验 
+--------------------------------+
```

# pause配置方式

```bash
ethtool -A <dev> tx <on/off> # 启用/禁用TX的流控
ethtool -A <dev> rx <on/off> # 启用/禁用RX的流控
```

# pfc帧格式

```bash
+--------------------------------+
|     Destination Address        |   // 6B, 目的MAC地址, 取值固定为 01-80-c2-00-00-01
+--------------------------------+
|     Source Address             |   // 6B, 源MAC地址
+--------------------------------+
|     Ethernet Type              |   // 2B, 以太网帧类型, 固定为 0x8808
+--------------------------------+
|     Control Opcode             |   // 2B, 控制码, 取值为 0x0001
+--------------------------------+
|     Priority Enable Vector     |   // 2B, 反压使能向量, 高字节为0, 低字节每bit对应一个优先级队列, n的取值范围为0~7
+--------------------------------+          每bit为1时表示对应的队列需要反压, 反压时间为Time(n), 为0表示不需要反压
|     Time(0~7)                  |   // 16B, 暂停时间, 即接收方请求发送方停止发送数据帧的时间长度
+--------------------------------+
|     Pad                        |   // 26B, 按以太网最小帧要求进行填充, 全 0 
+--------------------------------+
|     CRC                        |   // 4B, 循环冗余校验 
+--------------------------------+
```

# pfc反压

设备会为端口上的8个队列设置各自的PFC门限值，当队列已使用的缓存超过PFC门限值时，则向上游发送PFC反压通知报文，通知上游设备停止发包；当队列已使用的缓存降低到PFC门限值以下时，则向上游发送PFC反压停止报文，通知上游设备重新发包，从而最终实现报文的无丢包传输。

PFC中流量暂停只针对某一个或几个优先级队列，不针对整个接口进行中断，每个队列都能单独进行暂停或重启，而不影响其他队列上的流量，真正实现多种流量共享链路。而对非PFC控制的优先级队列，系统则不进行反压处理，即在发生拥塞时将直接丢弃报文。

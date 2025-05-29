# 什么是lacp

lacp，动态链路聚合，又称802.3ad。lacp协议创建一个聚合组，它们共享同样的速率和双工设定，多个slave工作在同一个激活的聚合体下。外出流量的slave选举是基于传输的hash策略，该策略可以通过xmit_hash_policy选项设置。

# 主从端口状态

Actor_state/Parter_state：（8bit）

1、LACP_Activity: 端口在链路控制中的主从状态，0表示Passive, 1表示Active。

2、LACP_Timeout: 超时时间，0表示长超时，1表示短超时。

3、Aggregation: 表示端口的聚合能力，TRUE(1)表示链路是可聚合的，FALSE(0)表示链路是独立链路，不可聚合。

4、Synchronization: 表示端口当前聚合是否完成。TRUE(1)表示发送的链路处于IN_SYNC状态，即端口已被分配到正确的聚合组中， FALS5(0)表示链路为OUT_OF_SYNC状态，即端口还没有选择正确的聚合组。

5、Collecting: TRUE(1)表示当前链路收包enable, 否则为FALSE(0)。

6、Distributing: TRUE(1)表示当前链路发包enable, 否则为FALSE(0)。

7、Defaulted： TRUE(1)表示Actor使用的Partner信息来自管理员配置的默认值。FALSE(0)表示Actor使用的Partner信息来自接收的LACDU。（用于调试）

8、Expired: TRUE(1)表示Actor RX状态机处于超时状态，否则不在超时状态。（用于调试）

# xmit_hash_policy参数取值

1、layer2：使用 2 层进行 hash ：（source mac XOR destionation mac）

2、layer2+3：使用 2 + 3 层进行 hash：（（source ip XOR destination ip） AND 0xffff）XOR（source mac XOR destination MAC）

3、layer3+4：使用 3 + 4 层进行 hash：（source prot XOR destination port）XOR（（source ip XOR destination ip） AND 0xffff）

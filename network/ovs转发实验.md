# 添加ovs配置命令的环境变量

```bash
export PATH=$PATH:/home/kayne/Public/ovs/install/sbin
export PATH=$PATH:/home/kayne/Public/ovs/install/bin
```

这里我的ovs编译后，安装在了
/home/kayne/Public/ovs/install目录下，并非/usr/local/bin目录，因此需要导入相关的环境变量。

# 启动ovs进程

```bash
mkdir -p /home/kayne/Public/ovs/install/var/run/openvswitch/
mkdir -p /home/kayne/Public/ovs/install/var/log/openvswitch/
mkdir -p /home/kayne/Public/ovs/install/etc/openvswitch/
ovsdb-tool create \
    /home/kayne/Public/ovs/install/etc/openvswitch/conf.db \
    /home/kayne/Public/ovs/install/share/openvswitch/vswitch.ovsschema
ovsdb-server \
    --remote=punix:/home/kayne/Public/ovs/install/var/run/openvswitch/db.sock \
    --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
    --private-key=db:Open_vSwitch,SSL,private_key \
    --certificate=db:Open_vSwitch,SSL,certificate \
    --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
    --pidfile --detach --log-file
ovs-vswitchd --pidfile --detach --log-file
```

# 创建ovs拓扑

```bash
+----------------------------+
|            ovs             |
+---+--------------------+---+
    |                    |
    |                    |
+---+---+            +---+---+
| veth1 |            | veth3 |
+---+---+            +---+---+
    |                    |
    |                    |
+---|------+      +------|---+
|   |      |      |      |   |
+---+---+  |      |  +---+---+
| veth0 |  |      |  | veth2 |
+-------+  |      |  +-------+
|          |      |          |
|   ns1    |      |    ns2   |
+----------+      +----------+
 sendp()             sniff()
```

```bash
ip netns add ns1
ip netns add ns2
ip link add veth0 type veth peer name veth1
ip link add veth2 type veth peer name veth3
ip link set veth0 netns ns1
ip link set veth2 netns ns2
ip netns exec ns1 ip link set veth0 up
ip link set veth1 up
ip netns exec ns2 ip link set veth2 up
ip link set veth3 up

ovs-vsctl add-br br0 -- set bridge br0 fail-mode=secure
ovs-vsctl add-port br0 veth1
ovs-vsctl add-port br0 veth3
```

# scapy发包

```bash
ip netns exec ns1 scapy
>>> sendp(Ether()/IP()/UDP()/'hello', loop=1, inter=1, iface='veth0')
```

# scapy收包

```bash
ip netns exec ns2 scapy
>>> sniff(filter='udp', iface='veth2', prn=lambda x: x.show())
```

在使用ovs-ofctl添加流表之前，是无法在veth2网卡上抓到包的，使用以下命令添加流表之后，就可以抓到包：

```bash
ovs-ofctl add-flow br0 in_port=veth1,action=output:veth3
```

由此可以看出，ovs打通了veth0网口（属于ns1命名空间）和veth2网口（属于ns2命名空间），使用ovs-ofctl查看流表命中情况：

```bash
# ovs-ofctl dump-flows br0
cookie=0x0, duration=2238.521s, table=0, n_packets=2216, n_bytes=104221, in_port=veth1, actions=output:veth3
```

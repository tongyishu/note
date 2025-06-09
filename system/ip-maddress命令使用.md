# 命令功能

ip maddress命令用于静态地管理链路层的组播地址（即MAC组播地址），对于协议层（即网络层，IPv4/IPv6）的组播地址，只能动态加入，无法进行手工配置。需要注意的是：

* 使用ip maddress add添加的静态条目，在使用ip maddress show查看时，都带有"static"字样的静态标识
* 不带"static"字样的条目，均为协议层动态添加的条目

# 使用方法

```bash
ip maddress [ add | del ] MULTIADDR dev NAME
ip maddress show [ dev NAME ]
```

# 使用示例

* 给eth0网卡添加组播地址33:33:00:00:00:01

```bash
ip maddress add 33:33:00:00:00:01 dev eth0
```

* 删除eth0网卡上的组播地址33:33:00:00:00:01

```bash
ip maddress del 33:33:00:00:00:01 dev eth0
```

* 查看eth0网卡上的组播地址

```bash
ip maddress show dev eth0
```

* 查看所有网卡上的组播地址

```bash
ip maddress show
```

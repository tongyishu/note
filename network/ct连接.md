# 什么是CT

连接跟踪，顾名思义就是跟踪并记录连接的状态。这里的连接可以是TCP连接，也可以是UDP连接。连接跟踪所做的事情就是发现并跟踪这些连接的状态：

1. 从数据包中提取元组（tuple）信息，辨别数据流（flow）和对应的连接（connection）。
2. 为所有连接维护一个状态数据库（conntrack table），例如连接的创建时间、发包数量、发包字节数等等。
3. 回收过期的连接（GC）。
4. 为更上层的功能（例如NAT）提供服务。

例如：

1. 拦截到一个TCP SYN包时，说明正在尝试建立TCP连接，需要创建一条新conntrack entry来记录这条连接。
2. 拦截到一个属于已有conntrack entry的包时，需要更新这条conntrack entry的收发包数等统计信息。
3. 拦截还要考虑性能问题，因为连接跟踪要对每个包进行过滤和分析

# 基于Netfilter的CT

提到连接跟踪，首先都会想到Netfilter。Netfilter是Linux内核中一个对数据包进行控制、修改和过滤的框架。它在内核协议栈中设置了若干hook点，以此对数据包进行拦截、过滤或其他处理。说的更直白一些，hook机制就是在数据包的必经之路上设置若干检测点，所有到达这些检测点的包都必须接受检测，根据检测的结果决定：

1. 放行：不对包进行任何修改，退出检测逻辑，继续后面正常的包处理。
2. 修改：例如修改IP地址进行NAT，然后将包放回正常的包处理逻辑。
3. 丢弃：安全策略或防火墙功能。

连接跟踪模块只是完成连接信息的采集和录入功能，并不会修改或丢弃数据包，后者是其他模块（例如NAT）基于Netfilter hook完成的。

需要说明的是：连接跟踪概念是独立于Netfilter的，Netfilter只是Linux内核中的一种连接跟踪实现的机制。换句话说，只要具备hook能力，就能拦截到进出主机的每个包，因此完全可以在此基础上自己实现一套连接跟踪。

# CT支持的协议

目前内核的连接跟踪支持以下六种协议：TCP、UDP、ICMP、DCCP、SCTP、GRE。不同版本的连接跟踪，支持的协议也不一样，但基本上所有的连接跟踪都支持TCP、UDP。

# 如何查看CT

以Linux内核为例，可以使用conntrack工具查看和修改连接跟踪。Conntrack为netfilter连接跟踪系统提供了一个全功能的用户空间接口，旨在取代旧的/proc/net/ip_conntrack接口。该工具用于搜索、遍历、检查和维护Linux内核的连接跟踪。

1、conntrack命令，常用选项

```bash
-L --dump
      列出所有的连接跟踪

-G, --get
      搜索并显示特定(匹配)条目

-D, --delete
      删除条目

-I, --create
      创建条目

-U, --update
      更新条目

-E, --event
      打印实时的event日志

-F, --flush
      清空条目

-C, --count
      统计条目个数

-S, --stats
      显示内核的连接跟踪统计信息
```

2、conntrack命令，使用举例

```bash
conntrack -L                                # 列出所有的连接跟踪
conntrack -L -p tcp --dport 80              # 查看目标端口为 80 的 TCP 连接
conntrack -L -s 192.168.1.100               # 查看源 IP 为 192.168.1.100 的连接
conntrack -L --state ESTABLISHED            # 查看所有已建立的连接
conntrack -F                                # 清空整个连接跟踪表
conntrack -E                                # 实时监控连接跟踪事件
conntrack -D -s 10.0.0.1 -p tcp --dport 22  # 删除源 IP 为 10.0.0.1 的 SSH 连接
```

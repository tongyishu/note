官方文档：
https://scapy.readthedocs.io/en/latest/usage.html

示例报文：
pkt = Ether() / IP() / UDP() / 'hello'

# get_if_list()

获取所有网络接口

```bash
>>> get_if_list()
['lo', 'enp0s3']
```

# get_if_addr()

获取网卡的IPv4地址

```bash
>>> get_if_addr('enp0s3')
'192.168.10.101'
```

# get_if_addr6()

获取网卡的IPv6地址

```bash
>>> get_if_addr6('enp0s3')
'240e:3af:e00:20d1:7234:c6ca:f6a0:9622'
```

# get_if_hwaddr()

获取网卡的MAC地址

```bash
>>> get_if_hwaddr('enp0s3')
'08:00:27:83:c9:a1'
```

# get_working_if()

获取生效的网口

```bash
>>> get_working_if()
'enp0s3'
```

# sendp()

发包函数，配置参数如下：

- iface 从指定端口发包
- loop 是否循环发包
- count 发送的报文个数
- inter 发包的时间间隔

```bash
>>> sendp(pkt, iface='enp0s3', loop=0, count=3, inter=1)
...
Sent 3 packets.
```

# sniff()

抓包函数，配置参数如下：

- iface 从指定的网口抓包
- prn 抓取的数据包的回调处理函数
- count 抓取的报文个数
- filter 过滤条件
- stop_filter 停止过滤的条件，指向一个回调函数，回调函数返回True时停止过滤

```bash
>>> sniff(iface='enp0s3', prn=lambda x: x.summary(), count=3, filter='tcp')
Ether / IP / TCP 192.168.10.111:52981 > 192.168.10.101:ssh A
Ether / IP / TCP 192.168.10.101:ssh > 192.168.10.111:52981 PA / Raw
Ether / IP / TCP 192.168.10.101:ssh > 192.168.10.111:52981 PA / Raw
<Sniffed: TCP:3 UDP:0 ICMP:0 Other:0>
```

# ls()

没有参数时可以列出scapy支持的所有的包的类型

```bash
>>> ls()
AH         : AH
AKMSuite   : AKM suite
ARP        : ARP
ASN1P_INTEGER : None
...
```

ls(pkt)打印指定包的内容

```bash
>>> ls(pkt)
dst        : DestMACField                        = 'ff:ff:ff:ff:ff:ff' (None)
src        : SourceMACField                      = '00:00:00:00:00:00' (None)
type       : XShortEnumField                     = 2048            (36864)
--
version    : BitField  (4 bits)                  = 4               (4)
ihl        : BitField  (4 bits)                  = None            (None)
tos        : XByteField                          = 0               (0)
len        : ShortField                          = None            (None)
id         : ShortField                          = 1               (1)
flags      : FlagsField  (3 bits)                = <Flag 0 ()>     (<Flag 0 ()>)
frag       : BitField  (13 bits)                 = 0               (0)
ttl        : ByteField                           = 64              (64)
proto      : ByteEnumField                       = 17              (0)
chksum     : XShortField                         = None            (None)
src        : SourceIPField                       = '127.0.0.1'     (None)
dst        : DestIPField                         = '127.0.0.1'     (None)
options    : PacketListField                     = []              ([])
--
sport      : ShortEnumField                      = 53              (53)
dport      : ShortEnumField                      = 53              (53)
len        : ShortField                          = None            (None)
chksum     : XShortField                         = None            (None)
--
load       : StrField                            = b'hello'        (b'')
```

# hexdump(pkt)

以十六进制的格式，打印报文内容

```bash
>>> hexdump(pkt)
0000  FF FF FF FF FF FF 00 00 00 00 00 00 08 00 45 00  ..............E.
0010  00 21 00 01 00 00 40 11 7C C9 7F 00 00 01 7F 00  .!....@.|.......
0020  00 01 00 35 00 35 00 0D BD 95 68 65 6C 6C 6F     ...5.5....hello
```

# bytes(pkt)

将包的内容转化为字节

```bash
>>> bytes(pkt)
b'\xff\xff\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00\x08\x00E\x00\x00!\x00\x01\x00\x00@\x11|\xc9\x7f\x00\x00\x01\x7f\x00\x00\x01\x005\x005\x00\r\xbd\x95hello'
```

# Raw(pkt)

以原生字节的形式查看报文，不作协议分层

```bash
>>> Raw(pkt)
<Raw  load='\xff\xff\xff\xff\xff\xff\x00\x00\x00\x00\x00\x00\x08\x00E\x00\x00!\x00\x01\x00\x00@\x11|\xc9\x7f\x00\x00\x01\x7f\x00\x00\x01\x005\x005\x00\r\xbd\x95hello' |>
```

# lsc()

列出所有的用户命令

```bash
>>> lsc()
IPID_count          : Identify IP id values classes in a list of packets
arpcachepoison      : Poison target's cache with (your MAC,victim's IP) couple
arping              : Send ARP who-has requests to determine which hosts are up
arpleak             : Exploit ARP leak flaws, like NetBSD-SA2017-002.
bind_layers         : Bind 2 layers on some specific fields' values.
bridge_and_sniff    : Forward traffic between interfaces if1 and if2, sniff and return
chexdump            : Build a per byte hexadecimal representation
computeNIGroupAddr  : Compute the NI group Address. Can take a FQDN as input parameter
...
```

# pkt.summary()

以摘要的形式查看报文

```bash
>>> pkt.summary()
'Ether / IP / UDP 127.0.0.1:domain > 127.0.0.1:domain / Raw'
```

# pkt.show()

协议分层的形式查看报文内容

```bash
>>> pkt.show()
###[ Ethernet ]### 
  dst= ff:ff:ff:ff:ff:ff
  src= 00:00:00:00:00:00
  type= IPv4
###[ IP ]### 
     version= 4
     ihl= None
     tos= 0x0
     len= None
     id= 1
     flags= 
     frag= 0
     ttl= 64
     proto= udp
     chksum= None
     src= 127.0.0.1
     dst= 127.0.0.1
     \options\
###[ UDP ]### 
        sport= domain
        dport= domain
        len= None
        chksum= None
###[ Raw ]### 
           load= 'hello'
```

# pkt.show2()

show2()的打印在格式上与show()没有区别。ls()/show()/show2()三者打印区别如下：

ls()与show()的共同点是：你构造的报文是什么样的，它们显示的就是什么样的。但最终发送出去的报文不一定和它们打印的一模一样。因为在报文发送之前，scapy会自动计算并填充ip_len，udp_len，checksum等字段。如果想要查看最终发送出去的报文长什么样，可以通过show2()打印，当然也可以通过抓包进行查看。

需要注意的是，如果在构造报文时指定了ip_len，udp_len，checksum等这些需要自动计算的字段，scapy是不会重新计算的。你指定了哪个字段，就使用你指定的那个字段的值，没有被指定的字段仍然会做自动计算。

# pkt.command()

查看报文生成的命令

```bash
>>> pkt.command()
"Ether()/IP()/UDP()/Raw(load=b'hello')"
```

# AsyncSniffer()

异步抓包的类，可以手动start/stop，上述sniff函数为同步抓包，该类与sniff()函数的参数类似

```bash
>>> sniffer = AsyncSniffer(iface='enp0s3')
>>> sniffer.start()
/usr/lib/python3/dist-packages/scapy/sendrecv.py:821: DeprecationWarning: setDaemon() is deprecated, set the daemon attribute instead
  self.thread.setDaemon(True)
>>> sniffer.stop()
<Sniffed: TCP:88 UDP:3 ICMP:0 Other:3>
```

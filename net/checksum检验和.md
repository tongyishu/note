# ipv4 checksum

校验范围：IP首部的20字节，不包括IP数据

```bash
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  0B
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  20B
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

发送端生成checksum：

1. **域段置零** : 将checksum字段设置为零，以便在计算中不受到影响
2. **拆分数据** : 将IPv4头部分成16位的短整数，如果头部的长度不是16位的倍数，填充0以确保长度为16位的倍数
3. **累加求和** : 对所有16位短整数执行无符号的加法，得到一个总和
4. **处理溢出** : 如果累加结果产生溢出（即结果超过16位），将溢出部分加回到结果的低16位
5. **按位取反** : 对最终的结果按位取反，得到checksum

接收端检验checksum:

1. **保存原始校验和** : 首先保存数据包中现有的校验和
2. **校验和字段置零** : 将头部中的校验和字段设置为零，以便重新计算
3. **重新计算校验和** : 按照校验和的计算方法，重新计算校验和
4. **比较两个校验和** : 比较重新计算的校验和与原始的校验和，如果两者一致，说明数据包头部未损坏

# tcp/udp checksum

校验范围：tcp/udp伪首部 + tcp/udp首部 + tcp/udp数据

计算方法：与IPv4校验和的计算方法类似

tcp/udp伪首部并非tcp/udp数据报中实际的有效成分。伪首部是一个虚拟的数据结构，其中的信息是从数据报所在IP分组头的分组头中提取的，既不向下传送也不向上递交，而仅仅是为计算校验和。

tcp/udp伪首部结构示意图：

```bash
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      Zeros    |      PTCL     |         TCP/UDP length        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* **Source Address** : 32bit，发送方的IP地址
* **Destination Address** : 32bit，接收方的IP地址
* **Zeros** : 8bit，保留字段，全置0
* **PTCL** : 8bit，protocol协议号，tcp为6，udp为17
* **TCP/UDP Length** : 16bit，tcp/udp报文的长度，包括tcp/udp首部和载荷

# icmp checksum

校验范围：icmp首部 + icmp数据，不包括IP首部

计算方法：与IPv4校验和的计算方法类似

# ipv6 checksum

ipv6首部没有checksum域段，但部分ipv6扩展头有checksum，比如icmpv6首部，icmpv6校验和的计算方法与ipv4的icmp报文类似：

* ipv6 header

  ```bash
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |Version| Traffic Class |           Flow Label                  |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |         Payload Length        |  Next Header  |   Hop Limit   |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     +                                                               +
     |                                                               |
     +                         Source Address                        +
     |                                                               |
     +                                                               +
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     +                                                               +
     |                                                               |
     +                      Destination Address                      +
     |                                                               |
     +                                                               +
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  ```
* icmpv6 header

```bash
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |     Type      |     Code      |          Checksum             |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                         Message Body                          +
      |                           ...                                 |
```

需要注意的是，ipv6下的tcp/udp报文，计算方法与ipv4下的类似，只不过伪头部不同：

```bash
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        TCP/UDP length                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                     Zeros                     |  Next Header  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

# checksum example

```python
from scapy.all import *


def ipv4_checksum(ipv4_hdr):
    if len(ipv4_hdr) % 2 != 0:
        ipv4_hdr += b'\x00'

    checksum = 0
    for i in range(0, len(ipv4_hdr), 2):
        word = (ipv4_hdr[i] << 8) + ipv4_hdr[i + 1]
        checksum += word

    while (checksum >> 16) > 0:
        checksum = (checksum & 0xFFFF) + (checksum >> 16)

    checksum = ~checksum & 0xFFFF
    return checksum


if __name__ == '__main__':
    pkt = IP(version=4,
             ihl=5,
             tos=0,
             len=128,
             id=1,
             flags=0,
             frag=0,
             ttl=64,
             proto="udp",
             src="127.0.0.1",
             dst="127.0.0.1")

    # scapy auto calculate, checksum=0x7c6a
    pkt.show2()

    # 1、init chksum=0
    pkt.chksum = 0
    # 2、manual calculate, checksum=0x7c6a
    checksum = ipv4_checksum(bytes(pkt))
    print(f'ipv4_checksum: {hex(checksum)}')

```

运行结果：

```bash
# python3 test.py
###[ IP ]###
  version   = 4
  ihl       = 5
  tos       = 0x0
  len       = 128
  id        = 1
  flags     =
  frag      = 0
  ttl       = 64
  proto     = udp
  chksum    = 0x7c6a
  src       = 127.0.0.1
  dst       = 127.0.0.1
  \options   \

ipv4_checksum: 0x7c6a

```
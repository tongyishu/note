GRE（Generic Routing Encapsulation，通用路由封装协议），对网络层协议（eg. ipv4、ipv6、appleTalk）的报文进行封装，**使这些被封装的报文能够在另一个网络层协议(eg. ipv4、ipv6、appleTalk)中传输** 。封装格式如下：

```bash
                  ---------------------------------
                  |                               |
                  |       Delivery Header         |
                  |                               |
                  ---------------------------------
                  |                               |
                  |       GRE Header              |
                  |                               |
                  ---------------------------------
                  |                               |
                  |       Payload packet          |
                  |                               |
                  ---------------------------------
```

* Delivery Header，投递协议（外层封装协议，eg. ipv4/ipv6）
* GRE Header，GRE头
* Payload packet，报文内容（内层乘客协议，eg. ipv4/ipv6）

# GRE Header

GRE Header在一开始比较复杂，其在rfc1701中的格式如下：

```bash
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |C|R|K|S|s|Recur|  Flags  | Ver |         Protocol Type         |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |      Checksum (optional)      |       Offset (optional)       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Key (optional)                        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                    Sequence Number (optional)                 |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Routing (optional)                    |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* C：1bit，Checksum存在位
* R：1bit，Routing存在位
* K：1bit，Key存在位
* S：1bit，Sequence Number存在位
* s：1bit，Strict Source Route
* Recur，3bit，递归控制位（Recursion Control），递归封装多少层，默认为0表示不递归
* Flags，5bit，标志位，保留为将来使用
* Ver，3bit，版本号（Version Number），必须为0
* Protocol Type，16bit，内层乘客协议的协议类型
* Offset，16bit，保留为将来使用
* Checksum，16bit，GRE头+负载的校验和
* Key，32bit，用于身份验证，确保数据包的来源合法
* Sequence Number，32bit，用于数据包保序
* Routing，32bit，长度可变，由一系列的**SREs** （Source Route Entries）组成

GRE Header在rfc2784中做了简化，简化后的格式如下：

```bash
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |C|       Reserved0       | Ver |         Protocol Type         |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      Checksum (optional)      |       Reserved1 (Optional)    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

由于简化的太严重又在rfc2890中做了补充。当前建议的GRE Header如下：

```bash
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |C| |K|S| Reserved0       | Ver |         Protocol Type         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |      Checksum (optional)      |       Reserved1 (Optional)    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Key (optional)                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                 Sequence Number (Optional)                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

# SRE

SRE（Source Route Entries），格式如下：

```bash
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |       Address Family          |  SRE Offset   |  SRE Length   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Routing Information ...                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* Address Family，16bit，指明了路由协议
* SRE Offset，8bit，SRE在Routing Information中的偏移
* SRE Length，8bit，SRE的个数
* Routing Information，长度可变，路由信息

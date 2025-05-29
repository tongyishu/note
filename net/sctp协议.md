# sctp业务场景

TCP Client和TCP Server之间的一条连接都只包含一个单一的流，这个架构存在的问题是：流中任何一点丢失，都会阻塞其数据传递。当传输文本的时候还可以接受，但是当传输的是实时数据（音视频）时就接受不了了。SCTP在每一条连接中提供多流服务（Multistream Service），这种连接在SCTP中称为关联（Association）。如果多个流中某一个被阻塞。其余的流仍然可以传递他们的数据。

# sctp架构设计

SCTP协议本身有Heartbeat机制来监控连接/路径的可用性。

SCTP两端都可以bind多个IP，因此一条SCTP连接可以采用不同的IP来传输。不同的IP对应不同的流，这些流都可以通过Heartbeat或者数据的传输/确认来监控其状态。

```bash
       _____________                                      _____________
      |  SCTP User  |                                    |  SCTP User  |
      | Application |                                    | Application |
      |-------------|                                    |-------------|
      |    SCTP     |                                    |    SCTP     |
      |  Transport  |                                    |  Transport  |
      |   Service   |                                    |   Service   |
      |-------------|                                    |-------------|
      |             |One or more    ----      One or more|             |
      | IP Network  |IP address      \/        IP address| IP Network  |
      |   Service   |appearances     /\       appearances|   Service   |
      |_____________|               ----                 |_____________|

        SCTP Node A |<-------- Network transport ------->| SCTP Node B

                         Figure 1: An SCTP Association
```

# sctp报文结构

sctp报文包含两部分，Common Header和Chunks：

```bash
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+           ---
       |     Source Port Number        |     Destination Port Number   |            ^
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+            |
       |                      Verification Tag                         |       Common Header
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+            |
       |                           Checksum                            |            v
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+           ---
       |                          Chunk #1                             |            ^
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+            |
       |                           ...                                 |          Chunks
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+            |
       |                          Chunk #n                             |            v
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+           ---
```

* Source Port Number，16bit，源端口号
* Destination Port Number，16bit，目的端口号
* Verification Tag，32bit，验证标签，用于报文验证
* Checksum，32bit，校验和

Chunk块的结构如下：

```bash
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |   Chunk Type  | Chunk  Flags  |        Chunk Length           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       \                                                               \
       /                          Chunk Value                          /
       \                                                               \
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* Chunk Type，8bit，块类型


| 类型编号                            | 解释说明                                                  |
| ------------------------------------- | ----------------------------------------------------------- |
| 0                                   | Payload Data (DATA)                                       |
| 1                                   | Initiation (INIT)                                         |
| 2                                   | Initiation Acknowledgement (INIT ACK)                     |
| 3                                   | Selective Acknowledgement (SACK)                          |
| 4                                   | Heartbeat Request (HEARTBEAT)                             |
| 5                                   | Heartbeat Acknowledgement (HEARTBEAT ACK)                 |
| 6                                   | Abort (ABORT)                                             |
| 7                                   | Shutdown (SHUTDOWN)                                       |
| 8                                   | Shutdown Acknowledgement (SHUTDOWN ACK)                   |
| 9                                   | Operation Error (ERROR)                                   |
| 10                                  | State Cookie (COOKIE ECHO)                                |
| 11                                  | Cookie Acknowledgement (COOKIE ACK)                       |
| 12                                  | Reserved for Explicit Congestion Notification Echo (ECNE) |
| 13                                  | Reserved for Congestion Window Reduced (CWR)              |
| 14                                  | Shutdown Complete (SHUTDOWN COMPLETE)                     |
| 15\~62，64\~126，128\~190，192\~254 | Available                                                 |
| 63，127，191，255                   | Reserved for IETF-defined Chunk Extension                 |

* Chunk Flags，8bit，块标志，取决于Type，不同的Chunk Type，Chunk Flags也不一样
* Chunk Length，16bit，块长度，为Type/Flags/Length/Value加在一起的总长度
* Chunk Value，块内容，取决于Type，不同的Chunk Type，Chunk Value的格式及内容也不一样

# sctp套接字

SCTP相关库参见gitee：
https://gitee.com/src-oepkgs/lksctp-tools.git

SCTP相关API详细参见：
https://docs.oracle.com/cd/E19120-01/open.solaris/817-4415/sockets-18/index.html

这里只列举部分SCTP的编程函数：

```c
int sctp_bindx(int sd, struct sockaddr *addrs, int addrcnt, int flags);
int sctp_connectx(int sd, struct sockaddr *addrs, int addrcnt, sctp_assoc_t *id);
int sctp_getpaddrs(int sd, sctp_assoc_t id, struct sockaddr **addrs);
int sctp_freepaddrs(struct sockaddr *addrs);
int sctp_getladdrs(int sd, sctp_assoc_t id, struct sockaddr **addrs);
int sctp_freeladdrs(struct sockaddr *addrs);
int sctp_sendmsg(int s, const void *msg, size_t len, struct sockaddr *to, socklen_t tolen, uint32_t ppid, uint32_t flags, uint16_t stream_no, uint32_t timetolive, uint32_t context);
int sctp_send(int s, const void *msg, size_t len, const struct sctp_sndrcvinfo *sinfo, int flags);
int sctp_recvmsg(int s, void *msg, size_t len, struct sockaddr *from, socklen_t *fromlen, struct sctp_sndrcvinfo *sinfo, int *msg_flags);
int sctp_getaddrlen(sa_family_t family);
```

# 基本字段


| **字段**     | **功能**                                                                                          |
| -------------- | --------------------------------------------------------------------------------------------------- |
| cookie       | 一个cookie标识一个flow，cookie=value或cookie=value/mask                                           |
| duration     | 流表生效时间，流表从下发到现在所持续的时间                                                        |
| table        | 流表所属表项，标识flow所属的表，默认为0                                                           |
| priority     | 流表优先级，范围为0~65535，值越大优先级越高                                                       |
| n_packets    | 流表匹配中的报文的个数                                                                            |
| n_bytes      | 流表匹配中的字节数                                                                                |
| idle_timeout | 流表空闲超时时间，流表会在空闲时间达到给的时间时被删除，设置为0（默认值）流表不会因空闲时间被删除 |
| hard_timeout | 流表可存在的时间，设置该值后，流表会在到达给的时间后被删除                                        |
| idle_age     | 流表空闲时间                                                                                      |
| hard_age     | 流表存在时间，该字段与duration字段的区别在于：当流表被修改后，会重置hard_age但是不会重置duration  |

# 匹配字段


| **字段**    | **功能**                                                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| in_port     | 匹配报文的入端口号（从哪个端口进来的）                                                                                                                                              |
| dl_vlan     | 匹配报文的vlan                                                                                                                                                                      |
| vlan_tci    | 匹配报文的vlan TCI                                                                                                                                                                  |
| dl_vlan_pcp | 匹配报文的vlan优先级                                                                                                                                                                |
| dl_type     | 匹配报文的二层协议类型：IPv4报文为0x0800，IPv6报文为0x86dd，ARP报文为0x0806                                                                                                         |
| dl_src      | 匹配报文的源MAC                                                                                                                                                                     |
| dl_dst      | 匹配报文的目的MAC                                                                                                                                                                   |
| nw_src      | 匹配报文的源IP                                                                                                                                                                      |
| nw_dst      | 匹配报文的目的IP                                                                                                                                                                    |
| nw_proto    | 匹配IP报文的协议类型，比如TCP/UDP/ICMP                                                                                                                                              |
| nw_tos      | 匹配IP报文的tos                                                                                                                                                                     |
| nw_ecn      | 匹配IP报文的ecn                                                                                                                                                                     |
| nw_ttl      | 匹配IP报文的ttl                                                                                                                                                                     |
| ip_frag     | 匹配IP报文的分片类型：no表示仅匹配非分片报文，yes表示匹配所有分片报文，first仅匹配offset为0的分片报文，later仅匹配offset非0的分片报文，not_later匹配非分片报文和offset为0的分片报文 |
| tp_src      | 匹配TCP/UDP报文的源端口                                                                                                                                                             |
| tp_dst      | 匹配TCP/UDP报文的目的端口                                                                                                                                                           |
| icmp_type   | 匹配ICMP报文的type                                                                                                                                                                  |
| icmp_code   | 匹配ICMP报文的code                                                                                                                                                                  |
| arp_sha     | 匹配ARP报文的源MAC                                                                                                                                                                  |
| arp_tha     | 匹配ARP报文的目的MAC                                                                                                                                                                |
| tun_id      | 匹配隧道报文的标识符                                                                                                                                                                |

# 动作字段


| **字段**                              | **功能**                                                                                                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| output                                | output:port 将报文从port端口发送output:src[start..end] 将报文从src读取的端口号发送，比如：output:NXM_NX_REG0[16..31]表示读取NXM_NX_REG0寄存器16~31位的数据作为端口号，并将报文从该端口发送 |
| enqueue                               | enqueue:port:queue 将报文送往port端口的queue队列                                                                                                                                           |
| normal                                | 将报文按照正常的L2/L3流程处理                                                                                                                                                              |
| flood                                 | 将报文发送到交换机上除接收端口和禁止flood端口外的所有端口                                                                                                                                  |
| all                                   | 将报文发送到队接收端口外的所有端口                                                                                                                                                         |
| controller                            | controller(key=value...) 将报文作为PACKET_IN消息发送到OpenFlow控制器                                                                                                                       |
| drop                                  | 丢弃报文                                                                                                                                                                                   |
| mod_vlan_vid                          | 修改vlan报文的vid                                                                                                                                                                          |
| mod_vlan_pcp                          | 修改vlan报文的优先级                                                                                                                                                                       |
| strip_vlan                            | 剥离报文的vlan                                                                                                                                                                             |
| push_vlan                             | 给报文添加vlan                                                                                                                                                                             |
| push_mpls                             | 给报文添加mpls标签                                                                                                                                                                         |
| pop_mpls                              | 去除报文的mpls标签                                                                                                                                                                         |
| mod_dl_src                            | 修改报文的源MAC                                                                                                                                                                            |
| mod_dl_dst                            | 修改报文的目的MAC                                                                                                                                                                          |
| mod_nw_src                            | 修改报文的源IP                                                                                                                                                                             |
| mod_nw_dst                            | 修改报文的目的IP                                                                                                                                                                           |
| mod_tp_src                            | 修改报文的源端口                                                                                                                                                                           |
| mod_tp_dst                            | 修改报文的目的端口                                                                                                                                                                         |
| mod_nw_tos                            | 修改报文的tos                                                                                                                                                                              |
| resubmit                              | resubmit(port,table) 使用port替换in_port字段并重新提交到table号表                                                                                                                          |
| set_tunnel                            | 修改隧道报文的tun_id                                                                                                                                                                       |
| set_queue                             | 设置报文的出队列                                                                                                                                                                           |
| pop_queue                             | 将队列恢复到使用set_queue设置之前的值                                                                                                                                                      |
| dec_ttl                               | 将报文的ttl减1                                                                                                                                                                             |
| set_mpls_ttl                          | 设置mpls报文的ttl                                                                                                                                                                          |
| dec_mpls_ttl                          | 将mpls报文的ttl减1                                                                                                                                                                         |
| push:src[start..end]                  | 将src中的start~end位表示的值压入内部堆栈                                                                                                                                                   |
| pop:dst[start..end]                   | 将内部堆栈顶部的值弹出到dst的start~end位                                                                                                                                                   |
| move:src[start..end]->dst[start..end] | 将src中的值写到dst，比如：move:NXM_NX_REG0[0..5]->NXM_NX_REG1[26..31]将NXM_NX_REG0寄存器的0~5位写到NXM_NX_REG1寄存器的26~31位                                                              |
| load:value->dst[start..end]           | 将value的值写入dst                                                                                                                                                                         |
| set_field:value->dst                  | 将value的值写入dst，dst需要指定名称，比如：set_field:16->ip_dscp                                                                                                                           |

注意：

由于历史原因，存在load和set_field两种操作方式。

openflow1.1引入了NXAST_REG_LOAD作为openflow1.0的Nicira扩展，并使用load来表示。

openflow1.2引入了一个标准的OFPAT_SET_FIELD操作，但在1.2的版本中set_field仅限于加载整个字段。

openflow1.5将OFPAT_SET_FIELD扩展到了NXAST_REG_LOAD超集的地步。

openvswitch会根据所使用的openflow版本转换两种语法：openflow1.1使用load; openflow1.2~1.4的版本兼容load和set_field; openflow1.5使用set_field。

# 什么是setpci

setpci命令用于读写pci设备的配置空间。pci设备的配置空间共有64字节，地址范围为0x00\~0x3f，这64字节是所有pci设备必须支持的。此外pcie设备还扩展了0x40\~0xff这段空间，该空间主要存放MSI/MSI-X中断和电源管理相关的Capability结构。

![](assets/20250317_222658_image.png)

需要注意的是：

* 该命令必须在root权限下使用
* 所有数字均采用十六进制表示

# 常用参数

-s 按Bus/Device/Function（总线/设备/功能编号）指定设备

-d 按Vendor/Device（供应商/设备ID）指定设备

-v 启用详细模式

b,w,l分别表示1,2,4字节。

# 使用示例

1、读取配置，其中设备的pci号00:1f.0，地址为0c，读取数据的位宽为1字节（b）

```bash
setpci -s 00:1f.0 0c.b
```

2、写入配置，其中设备的pci号00:bd.0，地址为68，写入数据的位宽为2字节（w），值为0xff

```bash
setpci -s 00:bd.0 68.w=ff
```

3、写入多个配置，其中设备的pci号为af:00.1，地址为0c的1字节（b）写入08，地址为54的4字节（l）写入0xffffffff

```bash
setpci -s af:00.1 0c.b=08 54.l=ffffffff
```

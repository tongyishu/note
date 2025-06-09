通过iso镜像文件制作qcow2镜像，本质上是一个安装操作系统的过程。iso类型是不能直接与qcow2类型互相转化的。

# 为什么要通过iso文件安装qcow2

iso文件的格式为iso9660，而iso9660是一个标准的CD-ROM文件系统，只可读不可写。因此需要将iso文件里面的内容提取出来，存放到qemu程序可识别的、可读写的qcow2格式的磁盘文件中，这个过程即为操作系统的安装过程。

# 需要准备的材料

一台装有qemu的物理服务器（不能是虚拟机）。

一份准备安装的操作系统的iso镜像文件（这里为CentOS-7-x86_64-DVD-1810.iso）。

一个VNC登录工具（这里为VncViewer.jar，对VncViewer的java封装）。

# 创建qcow2的磁盘文件

```bash
qemu-img create -f qcow2 CentOS-7-x86_64-DVD-1810.qcow2 20G
```

创建一个大小为20G的、文件系统为qcow2的磁盘镜像文件。

# 通过iso镜像文件拉起虚机

```bash
qemu-kvm \
  -m 4096 \
  -name CentOS-7-x86_64-DVD-1810 \
  -enable-kvm \
  -smp 4,sockets=4,cores=1,threads=1 \
  -drive file=CentOS-7-x86_64-DVD-1810.qcow2,format=qcow2 \
  -cdrom CentOS-7-x86_64-DVD-1810.iso \
  -net nic,macaddr=00:22:33:65:43:21 \
  -net tap,ifname=tap1,script=no,downscript=no \
  -boot c \
  -vnc 0.0.0.0:1
```

其中参数含义如下：

**-m** 内存大小

**-name** 要制作的镜像文件的名称

**-enable-kvm** 是否使用Linux内核的kvm.ko模块加速虚拟机

**-smp** 对称多处理，指定cpu插槽个数，每个cpu的核数，每个核的线程数

**-drive** 驱动文件，要制作的镜像文件及其格式

**-cdrom** 指定要安装的操作系统镜像，即cdrom类型的文件，这里为iso文件

**-net** 指定安装时配置的网卡

**-vnc** VNC地址，用于登录显示图形界面

**-boot** 定义设备的引导次序，每种设备使用一个字符表示。a、b 表示软驱，c表示第一块硬盘，d表示第一个光驱

# 通过VNC后台登录虚机

```bash
java -jar VncViewer.jar HOST 10.244.26.307 PORT 5901 SHAENCFLAG true
```

登录VNC，进入虚机操作系统的图形化界面进行安装，操作系统安装完成后，拷出qcow2，即为可用的虚拟磁盘文件。

**需要注意的是：-vnc 0.0.0.0:1对应此处的PORT 5901（5900 + 1）**

# 新建虚机

1、填写虚机名称、存放的文件夹、类型及版本。

![](assets/20250320_020140_image.png)

2、为虚机分配内存，这里我分配的内存为4个G。

![](assets/20250320_020207_image.png)

3、创建虚机磁盘。

![](assets/20250320_020233_image.png)

*这里我选择的是"VDI（VirtualBox磁盘映像）"。*

![](assets/20250320_020406_image.png)

*选择固定大小，防止后续磁盘不够用。*

![](assets/20250320_020428_image.png)

*这里我选择分配60G的磁盘。*

![](assets/20250320_020450_image.png)

*接下来就是等待磁盘创建完成。*

![](assets/20250320_020516_image.png)

# 注册操作系统镜像ISO文件

1、准备Linux的ISO镜像，这里我使用的是 CentOS-7-x86_64-DVD-1810.iso，可以从这个地址下载：
http://mirror.nsc.liu.se/centos-store/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso。

2、注册已经下载的ISO文件。

![](assets/20250320_020601_image.png)

![](assets/20250320_020620_image.png)

![](assets/20250320_020650_image.png)

![](assets/20250320_020710_image.png)

# 拉起虚机

1、点击启动开始起虚机。

![](assets/20250320_020729_image.png)

2、选择Install CenOS 7。

![](assets/20250320_020756_image.png)

![](assets/20250320_020821_image.png)

3、设置操作系统语言，这里我选择的English。

![](assets/20250320_020841_image.png)

4、选择安装的磁盘。

![](assets/20250320_020908_image.png)

5、选择已经创建的60G的磁盘。

![](assets/20250320_020931_image.png)

6、开始安装。

![](assets/20250320_020951_image.png)

7、在安装的过程中可以配置账号及密码，这里我配置的为root/root。

![](assets/20250320_021016_image.png)

![](assets/20250320_021034_image.png)

8、接下来等待操作系统安装完成reboot即可。

![](assets/20250320_021100_image.png)

9、这样的Linux虚机就安装好了，df -h可以查看磁盘分配。

![](assets/20250320_021122_image.png)

# brctl操作网桥

brctl操作系统不一定自带，需要自己安装bridge-utils包，常见的命令如下：

```bash
brctl addbr br0 # 添加br0网桥

brctl delbr br0 # 删除br0网桥

brctl addif br0 enp3s0 # 将enp3s0添加到br0网桥

brctl delif br0 enp3s0 # 将enp3s0从br0网桥上删除

brctl show br0 # 查看br0网桥的挂载情况

brctl stp br0 on # 打开br0网桥的stp功能

brctl stp br0 off # 关闭br0网桥的stp功能

brctl showstp br0 # 显示br0网桥的stp配置
```

# ip link操作网桥

ip link是当前操作系统主推的一款工具，十分全能，常用的命令如下：

```bash
ip link add br0 type bridge # 添加br0网桥

ip link del br0 type bridge # 删除br0网桥

ip link set dev enp3s0 master br0 # 将enp3s0添加至网桥br0上

ip link set dev enp3s0 nomaster # 将enp3s0从br0网桥上删除

ip link show enp3s0 # 可以查看enp3s0的网桥挂载情况，master后面的即为挂载的网桥
```

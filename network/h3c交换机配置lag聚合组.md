在H3C（华三V7）交换机上创建bond接口（又称聚合口或聚合组）通常使用以下命令：

# 1 创建聚合接口

```bash
system-view  # 切换到系统视图
interface Bridge-Aggregation 10  # 创建编号为10的bond口
```

其中"10"为要创建的bond口的编号，可以是任何未被使用的编号。

# 2 添加物理接口到聚合接口

```bash
interface GE1/0/57  # 切换到GE1/0/57的作用域
port link-aggregation group 10  # 将GE1/0/57添加到bond中
```

其中GE1/0/57为物理接口，该动作将GE1/0/57添加到之前创建的bond口中（编号为10的聚合口）。

# 3 配置聚合接口的参数

```bash
interface Bridge-Aggregation 10  # 切换到bond口的作用域
description "XXX"  # 配置描述信息
port link-type trunk  # 配置bond类型，access还是trunk
port default vlan 101  # 配置默认的vlan
```

这些命令用于配置bond口的描述信息、连接类型、以及默认VLAN）等参数。

# 4 启用聚合接口

```bash
interface Bridge-Aggregation 10
shutdown
undo shutdown
```

这些命令将先down掉bond口，再启用bond口。

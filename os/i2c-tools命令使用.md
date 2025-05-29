# i2c bus

I2C协议是一个允许多个 "从机" 芯片和一个多个的 "主机" 芯片进行通讯的协议。

I2C需要两根信号线完成信息交换，SCL时钟信号线，SDA数据输入/输出线（一般的计算机总线有三根：数据总线/控制总线/地址总线）。

I2C属于同步通信，由于输入输出数据均使用一根线，因此通信方向为半双工（能够双向通信，但需要分时操作）。

I2C所有组件之间都存在简单的主/从关系，连接到总线的每个设备均可通过唯一地址进行软件寻址。

I2C提供仲裁和冲突检测。

数据传输协议：

```bash
开始位   /   设备地址   /   读写位   /   响应   /   数据   /   响应   /   数据    /   响应   /   停止位
 1bit        7bit          1bit        1bit      8bit       1bit       8bit       1bit       1bit
```

# i2cdetect

探测i2c总线上的设备。

```bash
i2cdetect [-y] [-a] [-q|-r] I2CBUS [FIRST LAST]
i2cdetect -F I2CBUS
i2cdetect -l
```

**eg.** `i2cdetect -l`

**-y** 关闭交互模式

**-a** 扫描总线上的所有设备

**-q** 使用SMBus的quick write命令进行检测

**-r** 使用SMBus的receive byte命令进行检测

**I2CBUS** 指定i2c总线编号

**FIRST** 扫描的起始地址

**LAST** 扫描的终止地址

# i2cset

通过i2c总线设置寄存器的值。

```bash
i2cset [-f] [-y] [-m MASK] [-r] [-a] I2CBUS CHIP-ADDRESS DATA-ADDRESS [VALUE] ... [MODE]
```

eg. `i2cset -f -y 4 0x74 0x20 0xdf`

**-f** 强制访问设备

**-y** 关闭交互模式

**-f** 强制访问设备，即使设备已忙

**-y** 禁用交互模式，直接执行操作

**-m** mask 掩码参数，用于指定哪些位将被写入数据地址

**-r ** 写入值后立即回读，并将结果与写入的值进行比较

**-a** 自动标识偏移寄存器地址的设备

**i2cbus** I2C总线的编号

**CHIP-ADDRESS** I2C设备的地址

**DATA-ADDRESS** 要写入数据的寄存器地址

**VALUE** 要写入的数据值

**MODE** 指定数据的格式

* b (read byte data, default)
* w (read word data)
* s (read SMBus block data)
* i (read I2C block data, between 1 and 32, default 32)

# i2cget

通过i2c总线获取寄存器的值。

```bash
i2cget [-f] [-y] [-a] I2CBUS CHIP-ADDRESS DATA-ADDRESS [MODE]
```

**eg.** `i2cget -f -y 4 0x74 0x20`

**-f** 强制访问设备

**-y** 关闭交互模式

**I2CBUS** 指定i2c总线编号

**CHIP-ADDRESS** 设备的寄存器地址

**MODE** 指定读取的数据大小

* b (read byte data, default)
* w (read word data)
* s (read SMBus block data)
* i (read I2C block data, between 1 and 32, default 32)

# i2cdump

```bash
i2cdump [-f] [-y] [-r first-last] [-a] I2CBUS ADDRESS [MODE]
```

**eg.** `i2cdump -y 4 0x08`

**-f** 强制访问设备

**-y** 关闭交互模式

**I2CBUS** I2C总线的编号

**ADDRESS** dump的起始地址

**MODE** 指定读取的数据大小

* b (read byte data, default)
* w (read word data)
* s (read SMBus block data)
* i (read I2C block data, between 1 and 32, default 32)

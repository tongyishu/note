# **`ip link` 输出示例**

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
3: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000
    link/ether aa:bb:cc:dd:ee:ff brd ff:ff:ff:ff:ff:ff
```

# **逐字段解释**

## 1. 接口基本信息

* **`1: lo:`**
  * **`1`** ：接口的索引编号（系统分配的唯一标识）。
  * **`lo`** ：接口名称（`lo` 是本地环回接口）。

## 2. 接口状态标志

* **`<LOOPBACK,UP,LOWER_UP>`**
  表示接口的状态和特性（逗号分隔的多个标志）：
  * **常见标志** ：


    | 标志         | 含义                                                  |
    | -------------- | ------------------------------------------------------- |
    | `UP`         | 接口已启用（可通过`ip link set eth0 up/down` 开关）。 |
    | `LOWER_UP`   | 物理层连接正常（如网线已插好）。                      |
    | `NO-CARRIER` | 物理层无连接（如网线未插或损坏）。                    |
    | `BROADCAST`  | 支持广播通信（如以太网接口）。                        |
    | `MULTICAST`  | 支持组播通信。                                        |
    | `LOOPBACK`   | 环回接口（用于本地通信，如`127.0.0.1`）。             |
    | `SLAVE`      | 接口是绑定（bonding）或桥接（bridge）的从属接口。     |

## 3. MTU（最大传输单元）

* **`mtu 65536`**
  接口支持的最大数据包大小（单位：字节）。
  * 以太网默认 `1500`，环回接口（`lo`）通常为 `65536`。

## 4. 队列规则（qdisc）

* **`qdisc noqueue`**
  数据包调度算法（用于流量控制）：
  * **`noqueue`** ：无队列（直接处理，如环回接口）。
  * **`pfifo_fast`** ：默认的先进先出队列（FIFO）。
  * **`mq`** ：多队列（适用于多核 CPU 或高速网卡）。

## 5. 接口状态（state）

* **`state UP`**
  接口的逻辑状态：
  * **`UP`** ：接口已启用。
  * **`DOWN`** ：接口已禁用。
  * **`UNKNOWN`** ：状态未知（常见于虚拟接口如 `lo`）。
  * **`DORMANT`** ：接口处于休眠状态（如无线网卡未关联到网络）。

## 6. MAC 地址信息

* **`link/loopback 00:00:00:00:00:00`**
  * **`link/ether`** ：以太网接口的 MAC 地址。
  * **`link/loopback`** ：环回接口的虚拟 MAC 地址（固定为全零）。
  * **`brd ff:ff:ff:ff:ff:ff`** ：广播地址。

# **常见接口类型**

1. **物理接口** （如 `eth0`、`wlan0`）：
   * 名称通常以 `eth`（有线）、`wlan`（无线）开头。
   * 包含物理 MAC 地址（`link/ether xx:xx:xx:xx:xx:xx`）。
2. **虚拟接口** （如 `lo`、`docker0`）：
   * `lo`：本地环回接口，用于本机内部通信。
   * `docker0`：Docker 容器的默认虚拟网桥。

# TLS1.2协议支持DH（Diffie-Hellman）算法，该算法主要用于密钥协商

数学原理如下：

```bash
Ya = g ^ B mod p
Yb = g ^ A mod p

  =>

Ka = (Yb) ^ A mod p = (g ^ B mod p) ^ A mod p = g ^ (B * A) mod p
Kb = (Ya) ^ B mod p = (g ^ A mod p) ^ B mod p = g ^ (A * B) mod p

  =>

Ka = Kb
```

**Ka** 为Alice本地计算的对称加密的密钥。

**Kb** 为Bob本地计算的对称加密的密钥。

**p** 为大素数，Alice、Bob非加密传输协商确定。（公开）

**g** 为p的本原根，Alice、Bob非加密传输协商确定。（公开）

**A** 为Allice的私钥，仅Alice知道。（私有）

**B** 为Bob的私钥，仅Bob知道。（私有）

**Ya** 为Alice的公钥，经过非加密传输发送给Bob。（公开）

**Yb** 为Bob的公钥，经过非加密传输发送给Alice。（公开）

Alice与Bob协商后分别持有以下公开数据：

Alice: (p, g, A, Yb)

Bob: (p, g, B, Ya)

协商出来的对称密钥Ka =g ^ (B * A) mod p

协商出来的对称密钥Kb =g ^ (A * B) mod p

# 对称密钥Ka和Kb有以下特点

1、 Ka由Alice在本地计算获得，不会进行传输，不存在窃听风险。

2、 Kb由Bob在本地计算获得，不会进行传输，不存在窃听风险。

3、 Ka = Kb。

# DH算法破解的难度在于计算的不可逆性

已知 A，p，g 求 Ya 容易。

已知 Ya，p，g 求 A 难。

DH算法中的p，g被称为自定义域参数，可以通过openssl dhparam命令生成。

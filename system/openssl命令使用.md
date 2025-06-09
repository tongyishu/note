# 什么是openssl

OpenSSL是一个加密工具包，实现了安全套接字层(SSL v2/v3)和传输层安全(TLS v1)网络协议以及它们所需的相关加密标准。

openssl程序是一个命令行工具，用于从shell中使用openssl加密库的各种加密功能。它可以用来

* 创建和管理私钥、公钥和参数
* 公开密码匙加密操作
* 创建X.509证书、csr和crl
* 计算消息摘要
* 用密码加解密
* SSL/TLS客户端和服务器测试
* 处理S/MIME签署或加密邮件
* 时间戳请求、生成和验证

# 一级命令汇总

**asn1parse** 解析ASN.1序列

**ca** CA(Certificate Authority)管理

**ciphers** 加密套件

**cms** CMS(Cryptographic Message Syntax)工具

**crl** CRL(Certificate Revocation List)管理

**crl2pkcs7** CRL到PKCS#7的转换

**dgst** 消息摘要计算

**dh** Diffie-Hellman参数管理，已被dhparam所取代

**dhparam** Diffie-Hellman参数的生成与管理，已被genpkey和pkeyparam所取代

**dsa** DSA数据管理

**dsaparam** DSA参数生成和管理，已被genpkey和pkeyparam所取代

**ec** EC(Elliptic curve)椭圆曲线密钥处理

**ecparam** EC参数操作和生成

**enc** 使用加密套件进行加密

**engine** 引擎信息及操作

**errstr** 错误编号到错误字符串的转换

**gendh** Diffie-Hellman参数的生成，已被dhparam取代

**gendsa** 从参数生成DSA私钥，已被genpkey和pkey取代

**genpkey** 私钥或参数生成

**genrsa** RSA私钥生成，已被genpkey取代

**nseq** 创建或检查一个netscape证书序列

**ocsp** 在线证书状态协议实用程序

**passwd** 生成哈希密码

**pkcs12** PKCS#12数据管理

**pkcs7** PKCS#7数据管理

**pkey** 公私钥管理

**pkeyparam** 公钥算法参数管理

**pkeyutl** 公钥算法加密操作实用程序

**rand** 生成伪随机字节

**req** PKCS#10 X.509证书签名请求(CSR)管理

**rsa** rsa密钥管理

**rsautl** RSA实用程序，用于签名、验证、加密和解密，已被pkeyutl取代

**s_client** 实现了一个通用的SSL/TLS客户端，它可以和SSL/TLS远程服务器建立连接，它仅用于测试目的，并只提供基本的接口功能，但在内部使用OpenSSL ssl库的大部分功能

**s_server** 实现了一个通用的SSL/TLS服务器，它可以接受SSL/TLS远程客户端的连接，它仅用于测试目的，并只提供基本的接口功能，但在内部使用OpenSSL ssl库的大部分功能，它为测试SSL功能提供了面向命令行的协议，以及一个简单的HTTP响应工具

**s_time** SSL连接定时器

**sess_id** SSL会话数据管理

**smime** S/MIME邮件处理

**speed** 算法速度测量

**spkac** SPKAC打印和生成实用程序

**ts** 时间戳授权工具(客户端/服务器)

**verify** X.509证书验证

**version** OpenSSL版本信息

**x509** X.509证书数据管理

# 生成私钥

```bash
openssl genrsa -aes256 -out rsa.key 2048
```

-aes256 使用aes256算法加密私钥

-out 输出私钥到文件

2048 私钥长度为2048bits

# 提取公钥

```bash
openssl rsa -in rsa.key -pubout -out rsa_pub.key
```

-in 输入的私钥文件

-pubout 表明要输出的密钥文件为公钥

-out 输出公钥到文件

# 公钥加密文件

```bash
openssl rsautl -encrypt -in data.txt -inkey rsa_pub.key -pubin -out data.txt.encrypt
```

-encrypt 执行加密操作

-in 要加密的文件

-inkey 加密使用的密钥（公钥）

-pubin 表明输入的密钥为公钥

-out 加密后的文件

# 私钥解密文件

```bash
openssl rsautl -decrypt -in data.txt.encrypt -inkey rsa.key -out data.txt
```

-decrypt 执行解密操作

-in 要解密的文件

-inkey 解密使用的密钥（私钥）

-out 解密后的文件

# 私钥签名文件

```bash
openssl dgst -sha256 -sign rsa.key -out data.txt.sign data.txt
```

-sha256 使用sha256算法生成摘要

-sign 签名使用的私钥文件

-out 生成的签名文件

data.txt 要进行签名的文件

# 公钥验签文件

```bash
openssl dgst -sha256 -verify rsa_pub.key -signature data.txt.sign data.txt
```

-sha256使用sha256算法生成摘要

-verify 验签使用的公钥

-signature 验签使用的的公钥

data.txt 要验签的文件

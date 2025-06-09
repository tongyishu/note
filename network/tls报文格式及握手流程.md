# TLS 报文头格式

TLS 报文头共 5 字节，第 1 字节是类型（目前有 4 种类型），第 2-3 字节是版本（目前有 4 种版本），第 4-5 字节是长度（不包含报文头本身长度）。

- TLS Record Header 5B

```bash
TLS Record Header 5B
+------+---------------+------------+
| Type |     Version   |   Length   |
| (1B) | (Major,Minor) |   (2B)     |
|      |     (2B)      |            |
+------+---------------+------------+
```

Record Type Values (1B):


| Type               | Dec | Hex  |
| -------------------- | ----- | ------ |
| CHANGE_CIPHER_SPEC | 20  | 0x14 |
| ALERT              | 21  | 0x15 |
| HANDSHAKE          | 22  | 0x16 |
| APPLICATION_DATA   | 23  | 0x17 |

Version Values (2B, major.minor in hex):


| Protocol | Dec | Hex    |
| ---------- | ----- | -------- |
| SSL 3.0  | 3,0 | 0x0300 |
| TLS 1.0  | 3,1 | 0x0301 |
| TLS 1.1  | 3,2 | 0x0302 |
| TLS 1.2  | 3,3 | 0x0303 |

# Handshake 握手过程

```bash
+----------+                             +----------+
|  CLIENT  |                             |  SERVER  |
+----------+                             +----------+
     |                                       |
     |          ClientHello                  |
     | o-----------------------------------> |
     |                                       |
     |          ServerHello                  |
     |          Certificate                  |
     |          ServerKeyExchange            |
     |          CertificateRequest           |
     |          ServerHelloDone              |
     | <-----------------------------------o |
     |                                       |
     |          Certificate                  |
     |          ClientKeyExchange            |
     |          CertificateVerify            |
     |          ChangeCipherSpec             |
     |          Finished                     |
     | o-----------------------------------> |
     |                                       |
     |          ChangeCipherSpec             |
     |          Finished                     |
     | <-----------------------------------o |
     |                                       |
```

1. 客户端发送 ClientHello 报文。其主要作用是告诉服务器，本客户端所支持的 TLS 协议版本，以及所支持的加密算法等等。
2. 服务器发送 ServerHello 报文。其主要作用是服务器选择一个它认为安全的，且双方都支持的加密算法；如果服务器认为客户端所有支持的加密算法都不安全，则服务器可以发送一个 ALERT 报文。
3. 服务器发送 Certificate 报文。其主要作用是服务器发送自己的证书给客户端。
4. 服务器发送 ServerKeyExchange 。其主要作用是提供一些信息，以便双方有足够的信息来约定一个数据传输阶段所使用的对称密钥算法的密钥。这个报文是可选的，如果使用 Diffie-Hellman 方式来约定密钥，则这个是必须的；如果是 RSA 方式来约定密钥，它可以省略。
5. 服务器发送 CertificateRequest 。其作用是开启“双向认证（Mutual authentication）”模式，即不仅客户端要验证服务器，而且服务器还要验证客户端。这种方式在 https 网站中很少使用，如果对 https 网站进行抓包分析，一般都不会有这个报文。
6. 服务器发送 ServerHelloDone，其作用是告诉客户端。
7. 客户端发送 Certificate 报文（仅当客户端收到了 CertificateRequest 时才发送，即服务器开启了双向认证）。其主要作用是客户端发送自己的证书给服务器。
8. 客户端发送 ClientKeyExchange 报文。其主要作用是提供一些信息，以便双方有足够的信息来约定一个数据传输阶段所使用的对称密钥算法的密钥。如果是 RSA 方式，则客户端生成一个对称密钥算法的密钥后，使用服务器的公钥进行加密后传送给服务器。如果是 Diffie-Hellman 方式，则传送必要信息以便双方可以按约定方式生成同一个密钥。
9. 客户端发送 CertificateVerify 报文（仅当客户端收到了 CertificateRequest 时才发送，即服务器开启了双向认证）。主要作用是客户端发送一段它签名的信息给服务器，这样服务器使用客户端的公钥就可以验证签名，从而验证客户端。
10. 客户端发送 ChangeCipherSpec 报文，告诉服务器你可以使用加密模式了。注： ChangeCipherSpec 报文不属于 Handshake 报文，它是 TLS 顶级报文。
11. 客户端发送 Finished 报文，告诉服务器我准备好加密通信了。
12. 服务器发送 ChangeCipherSpec 报文，告诉客户端你可以使用加密模式了。
13. 服务器发送 Finished 报文，告诉客户端我准备好加密通信了，至此握手结束。

# Handshake 报文格式

```bash
   Record Layer (5B)             Handshake Layer (Variable)
+---------------------+-----------------------------------------------+
| Type | Version| Len | Handshake Type | Length | Handshake Message   |
| (1B) |  (1B)  |(2B) | (1B)           | (3B)   | (up to 16KB)        |
+---------------------+-----------------------------------------------+
```

共有 10 种类型的 Handshake 报文：


| Handshake Type      | Dec | Hex  |
| --------------------- | ----- | ------ |
| HELLO_REQUEST       | 0   | 0x00 |
| CLIENT_HELLO        | 1   | 0x01 |
| SERVER_HELLO        | 2   | 0x02 |
| CERTIFICATE         | 11  | 0x0B |
| SERVER_KEY_EXCHANGE | 12  | 0x0C |
| CERTIFICATE_REQUEST | 13  | 0x0D |
| SERVER_HELLO_DONE   | 14  | 0x0E |
| CERTIFICATE_VERIFY  | 15  | 0x0F |
| CLIENT_KEY_EXCHANGE | 16  | 0x10 |
| FINISHED            | 20  | 0x14 |

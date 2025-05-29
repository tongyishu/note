# 什么是protobuf?

Google Protocol Buffer，简称protobuf或GPB，是谷歌开发的一种语言中立、平台中立且可扩展的结构化数据序列化语言，广泛用于通信协议、数据存储等领域。与之类似的序列化语言还有JSON，只不过JSON使用纯文本定义和存储数据，protobuf使用.proto文本文件定义数据格式，使用二进制文件存储具体的数据。

由于protobuf由于二进制格式，数据更紧凑，占用空间更少，解析速度更快。但其易读性不如JSON，需要经过特定的工具"翻译"，而JSON则是所见即所得。

# protobuf示例

1、使用.proto文件定义一个tree结构体，包括root，leaf两个属性。

```bash
syntax = "proto3";

message Tree {
  int root = 1;
  int leaf = 2;
}
```

2、使用protoc-c工具生成相关.h和.c文件，这些文件用于业务代码进行序列化和反序列化操作：

```bash
protoc-c tree.proto --c_out=.
```

以上命令会生成tree.pb-c.h和tree.pb-c.c文件，这两个文件定义tree结构体的序列化和反序列化的方法，并提供了c语言形式的接口API，用于操作该结构体。

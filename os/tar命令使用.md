# 什么是归档

归档（Archive）是指将多个文件和目录打包成一个单独的文件，这个过程通常不涉及压缩，只是简单地将文件和目录组织在一起，形成一个逻辑上的整体。

# 什么是压缩率

压缩率是指文件压缩前大小与压缩后大小之比。压缩率越大，说明压缩程度越高，即原始数据经过压缩后所占用的空间更小。

# 常用的归档和压缩工具

windows：rar，zip，7z

linux：tar，cpio，ar，xz，gzip，bzip2

需要注意的是，linux上的tar/cpio/ar为归档工具，xz，gzip，bzip2为压缩工具。

值得一提的是，linux上的xz和windows上的7z使用的压缩算法相同，均为LZMA2算法（7z也可以指定LZMA算法），压缩率非常高。

# tar的几种压缩方式

tar命令是一个归档命令，本身并不具备压缩能力，但可以和其它压缩工具一起使用（比如Linux上常用的有：xz，gzip，bzip2），用来归档和压缩文件。

* tar -z，使用gzip进行压缩
* tar -j，使用bzip2进行压缩
* tar -J，使用xz进行压缩

使用举例：

```bash
tar -czvf xxx.tar.gz a.txt b.txt
tar -xzvf xxx.tar.gz

tar -cjvf xxx.tar.bz a.txt b.txt
tar -xjvf xxx.tar.bz

tar -cJvf xxx.tar.xz a.txt b.txt
tar -xJvf xxx.tar.xz
```

# gz/bz/xz性能比较


| 压缩方式 | 压缩率 | 压缩速度 | 解压速度 | 压缩算法 |
| ---------- | -------- | ---------- | ---------- | ---------- |
| gz       | 低     | 快       | 快       | DEFLATE  |
| bz       | 中     | 慢       | 中       | BWT      |
| xz       | 高     | 中       | 慢       | LZMA2    |

gzip适用于耗时较少的场景，xz适用于空间占用较小的场景，如果需要平衡空间和时间，可以考虑两方面表现均适中的bzip2。

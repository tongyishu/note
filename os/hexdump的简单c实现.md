收发包的过程中，经常要打印报文内容来检查收到或者发送的包是否正确。比如tcpdump工具的-X选项，能够以16进制的形式打印报文内容，显示对应的ASCII字符。

```bash
# tcpdump -i ens2f0 -X
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens2f0, link-type EN10MB (Ethernet), capture size 262144 bytes
17:22:54.399494 IP 20-165-33.ssh > 10.1.90.46.51036: Flags [P.], seq 349080497:349080565, ack 3808226675, win 2122, length 68
        0x0000:  4548 006c 6406 4000 4006 c2d8 0a15 a521  EH.ld.@.@......!
        0x0010:  0a01 5a2e 0016 c75c 14ce 8bb1 e2fc ed73  ..Z....\.......s
        0x0020:  5018 084a 13c4 0000 be64 765f 8554 321b  P..J.....dv_.T2.
        0x0030:  f576 d6f5 fc1a fb2b 7fc1 0467 3f8e db68  .v.....+...g?..h
        0x0040:  7900 a404 aa8d 3ad5 4d95 4a53 16d6 4fbe  y.....:.M.JS..O.
        0x0050:  5a56 25a3 11bf 6cf4 96c1 1f7c f1d7 03bb  ZV%...l....|....
        0x0060:  2809 e496 0302 0435 43ca 25c2            (......5C.%.
```

与之类似的工具有很多，再比如Linux操作系统自带的hexdump工具，可以打印显示二进制文件的内容：-C,--canonical按规范打印，-n,--length 指定打印的字节个数。

```bash
# hexdump -C -n 128 /dev/urandom
00000000  69 18 c4 1e e8 68 de 6f  6a 83 47 1f 2c 03 8d 13  |i....h.oj.G.,...|
00000010  c0 bf a6 77 7f 55 5b fd  fb 6b 34 8b 31 32 84 3d  |...w.U[..k4.12.=|
00000020  99 9b e3 1e 78 da f4 43  19 5c 55 e0 1c ca 75 35  |....x..C.\U...u5|
00000030  12 62 54 1f 1e e3 c4 e3  f5 72 55 80 d6 9c c2 de  |.bT......rU.....|
00000040  d2 53 7c 0b ef 35 0f 58  40 d8 eb c3 c4 ae 6a 39  |.S|..5.X@.....j9|
00000050  b1 15 2a 4d 9d b8 08 0b  ba ff 0f 82 2e a1 ce 62  |..*M...........b|
00000060  c4 2d 5e 70 7c 31 ae 3d  95 10 2c 87 36 bc be 38  |.-^p|1.=..,.6..8|
00000070  dc 24 ce ae 26 31 18 d3  e5 58 79 a4 98 34 27 e7  |.$..&1...Xy..4'.|
00000080
```

这些工具大都实现比较复杂，不好直接将其移植到自己的代码中，二进制工具也不好在C码中直接调用。下面给出一个简单的hex_dump的C码实现，可以直接拷贝使用：

```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

void hex_dump(unsigned char *buf, unsigned int buf_len)
{
#define VISIABLE(ch) (unsigned int)(((ch) - ' ') < (127u - ' '))

	for (int i = 0; i < buf_len; i += 16) {
		printf("%08X: ", i);

		for (int j = 0; j < 16; j++) {
			if (i + j < buf_len)
				printf("%02X ", buf[i + j]);
			else
				printf("   ");
		}
		printf("   ");

		for (int j = 0; j < 16; j++) {
			if (i + j < buf_len)
				printf("%c", VISIABLE(buf[i + j]) ? buf[i + j] : '.');
		}
		printf("\n");
	}
}

int main(int argc, char *argv[])
{
#define BUF_LEN 128
	int fd;
	char buf[BUF_LEN] = {0};
	ssize_t ret;
	ssize_t cnt;

	// 读取随机数
	fd = open("/dev/urandom", O_RDONLY);
	if (fd == -1)
		return -1;

	cnt = 0;
	while (cnt < BUF_LEN) {
		ret = read(fd, buf + cnt, BUF_LEN - cnt);
		if (ret < 0) {
			close(fd);
			return 1;
		}
		cnt += ret;
	}

	// 打印随机数
	hex_dump(buf, BUF_LEN);
	close(fd);
	return 0;
}
```

输出结果如下：

```bash
# gcc -o hex_dump hex_dump.c && ./hex_dump
00000000: 2B CF 83 91 F5 ED 91 D1 35 8E 40 E9 48 42 E4 03    +.......5.@.HB..
00000010: 9C 28 28 F3 96 C0 D4 4C 79 BC 2F 31 79 F8 64 6C    .((....Ly./1y.dl
00000020: A5 39 A3 70 B9 FB A2 46 5A 77 79 4B F3 28 80 2E    .9.p...FZwyK.(..
00000030: 5B EE 44 1A 58 61 25 EA 24 9F 8A B2 A4 54 F6 DC    [.D.Xa%.$....T..
00000040: 26 CA A2 66 70 62 C9 B6 39 51 64 63 C3 57 57 86    &..fpb..9Qdc.WW.
00000050: C6 1E F9 27 C8 CF EF 3F 4D 6B 8C 89 29 BC 83 B1    ...'...?Mk..)...
00000060: 16 1B 40 54 5B 85 95 C0 7D F8 15 BE F0 61 12 BD    ..@T[...}....a..
00000070: 1B B4 51 FD 15 95 C1 E4 77 0B FE 5E E9 28 15 DF    ..Q.....w..^.(..
```

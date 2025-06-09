* 宏NLMSG_ALIGN(len)用于得到不小于len且字节对齐的最小数值

```c
#define NLMSG_ALIGNTO 4U
#define NLMSG_ALIGN(len) (((len)+NLMSG_ALIGNTO-1) & ~(NLMSG_ALIGNTO-1))
```

* 宏NLMSG_HDRLEN用于计算nlmsghdr消息头的长度

```c
#define NLMSG_HDRLEN ((int) NLMSG_ALIGN(sizeof(struct nlmsghdr)))
```

* 宏NLMSG_LENGTH(len)用于计算数据部分长度为len时实际的消息长度

```c
#define NLMSG_LENGTH(len) ((len) + NLMSG_HDRLEN)
```

* 宏NLMSG_SPACE(len)返回不小于NLMSG_LENGTH(len)且字节对齐的最小数值

```c
#define NLMSG_SPACE(len) NLMSG_ALIGN(NLMSG_LENGTH(len))
```

* 宏NLMSG_DATA(nlh)用于取得消息的数据部分的首地址

```c
#define NLMSG_DATA(nlh) ((void*)(((char*)nlh) + NLMSG_LENGTH(0)))
```

* 宏NLMSG_NEXT(nlh,len)用于得到下一个消息的首地址，同时len也减少为剩余消息的总长度
```c
#define NLMSG_NEXT(nlh,len) ((len) -= NLMSG_ALIGN((nlh)->nlmsg_len), (struct nlmsghdr*)(((char*)(nlh)) + NLMSG_ALIGN((nlh)->nlmsg_len)))
```

* 宏NLMSG_OK(nlh,len)用于判断消息是否有len这么长

```c
#define NLMSG_OK(nlh,len) ((len) >= (int)sizeof(struct nlmsghdr) && (nlh)->nlmsg_len >= sizeof(struct nlmsghdr) && (nlh)->nlmsg_len <= (len))
```

* 宏NLMSG_PAYLOAD(nlh,len)用于返回payload的长度

```c
#define NLMSG_PAYLOAD(nlh,len) ((nlh)->nlmsg_len - NLMSG_SPACE((len)))
```

使用netlink获取网卡接口名称的代码如下：

```c
#include <linux/netlink.h>   // struct sockaddr_nl
#include <linux/rtnetlink.h> // struct rtgenmsg, struct ifinfomsg
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h> // struct msghdr
#include <unistd.h>

#define BUFSIZE 8192

void rtnl_print_link(struct nlmsghdr *h)
{
	struct ifinfomsg *iface = NLMSG_DATA(h);
	int len = RTM_PAYLOAD(h);

	for (struct rtattr *attr = IFLA_RTA(iface); RTA_OK(attr, len); attr = RTA_NEXT(attr, len)) {
		switch (attr->rta_type) {
		case IFLA_IFNAME:
			printf("Interface %d: %s\n", iface->ifi_index, (char *)RTA_DATA(attr));
			break;

		default:
			break;
		}
	}
}

int main(int argc, char *argv[])
{
	struct sockaddr_nl sock = {
	    .nl_family = AF_NETLINK,
	};

	int fd = socket(AF_NETLINK, SOCK_RAW, NETLINK_ROUTE);
	if (fd < 0)
		return -1;

	struct nlmsghdr hdr = {
	    .nlmsg_len = NLMSG_LENGTH(sizeof(struct rtgenmsg)),
	    .nlmsg_type = RTM_GETLINK,
	    .nlmsg_flags = NLM_F_REQUEST | NLM_F_DUMP,
	    .nlmsg_seq = 1,
	    .nlmsg_pid = getpid(),
	};

	struct iovec io = {
	    .iov_base = &hdr,
	    .iov_len = hdr.nlmsg_len,
	};

	struct msghdr msg = {
	    .msg_iov = &io,
	    .msg_iovlen = 1,
	    .msg_name = &sock,
	    .msg_namelen = sizeof(sock),
	};

	/* sendmsg: 1 nlmsghdr, RTM_GETLINK
	   +-----------------+---------+
	   | struct nlmsghdr | payload |
	   +-----------------+---------+
	*/
	if (sendmsg(fd, &msg, 0) < 0)
		return -1;

	int done = 0;
	char buf[BUFSIZE];

	/* recvmsg: N nlmsghdr, 0 ~ N-2 is RTM_NEWLINK, N-1 is NLMSG_DONE
	   +-----------------+---------------+-----------------+---------------+-----------------+
	   | struct nlmsghdr |    payload    | struct nlmsghdr |    payload    |      ...        |
	   +-----------------+---------------+-----------------+---------------+-----------------+
	                     <- NLMSG_DATA ->                  <- NLMSG_DATA ->
	   <--------------------------- N (struct nlmsghdr + payload)---------------------------->
	*/
	while (done == 0) {
		memset(buf, 0, sizeof(buf));
		msg.msg_iov->iov_base = buf;
		msg.msg_iov->iov_len = sizeof(buf);

		int len = recvmsg(fd, &msg, 0);
		if (len < 0)
			return -1;

		for (struct nlmsghdr *mp = (struct nlmsghdr *)buf; NLMSG_OK(mp, len);
		     mp = NLMSG_NEXT(mp, len)) {
			switch (mp->nlmsg_type) {
			case NLMSG_DONE:
				done++;
				break;
			case RTM_NEWLINK:
				rtnl_print_link(mp);
				break;
			default:
				printf("Ignore msg: type=%d, len=%d\n", mp->nlmsg_type,
				       mp->nlmsg_len);
				break;
			}
		}
	}
	close(fd);
	return 0;
}
```

运行结果如下：

```bash
# gcc -o test test.c && ./test
Interface 1: lo
Interface 2: enp175s0f0
Interface 3: enp175s0f1
Interface 4: docker0
Interface 6: vethf93889b
Interface 8: veth1c7549b
```
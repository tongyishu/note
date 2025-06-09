# getopt用于解析shell程序的命令行参数，它后面紧跟的参数分为两部分

1、getopt命令本身的参数。

2、要解析的命令行参数。

通常这两部分会用"--"隔开，如果没有"--"，第二部分将从第一个非getopt选项参数的位置开始。如果第一部分中没有'-o'或'--options'选项，则使用第二部分的第一个参数作为短选项字符串。

# 一个例子说明getopt的使用

```bash
function usage() {
	echo "
usage:
	-i, --ip    server ip
	-p, --port  server port
	-h, --help  display this help and exit
"
}

function parse_args() {
	args=`getopt -o i:p::h -l ip:,port::,help -- $@`
	set -- $args

	while true; do
		case $1 in
			-i|--ip)
				IP=$2
				shift 2
				;;
			-p|--port)
				PORT=$2
				shift 2
				;;
			-h|--help)
				usage
				exit 1
				;;
			--)
				shift 1
				;;
			*)
				if test "x$1" == "x"; then
					break
				else
					echo "getopt: unrecongnized option $1"
					exit 1
				fi
				;;
		esac
	done
}

IP=
PORT=
parse_args $@
echo "server ip: $IP"
echo "server port: $PORT"

```

# 使用示例

```bash
# ./test.sh --help

usage:
        -i, --ip    server ip
        -p, --port  server port
        -h, --help  display this help and exit
```

```bash
# ./test.sh --ip=192.168.1.1 --port=2345
server ip: '192.168.1.1'
server port: '2345'
```

# 以"i:p::h"和"ip:,port::,help"为例作解释说明

**i:** 短选项和 **ip:** 长选项，后面跟一个":"，表示该选项后必带值。

**p::** 短选项和 **port::** 长选项，后面跟两个"::"，表示该选项后是可选值。

**h** 短选项和 **help** 长选项，后面没有跟":"，表示该选项后不带值。

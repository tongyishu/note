# 为什么需要getopts

解析shell的入参是一件让人头疼的事，因为要从`$@`或`$*`来读取入参，并不断地shift来移位，还要配合`$1`的使用才能完成参数的解析。实际上bash在设计时就考虑到了这种情况，并提供了`getopts`内置命令用于解析入参。

`getopts`有两个参数，第一个参数是一个字符串，包括字符和":"，其中每一个字符都是一个有效的option。如果字符后面带有":"，表示这个选项有自己的argument，argument保存在内置变量`OPTARG`中。

而`$OPTARG`总是存储原始`$*`中下一个要处理的元素位置。

# 一个例子展示getopts

一个例子展示getopt的使用：

```bash
#!/bin/bash
BUILD=0
RPMBUILD=0
UPLOAD=0
CLEAN=0
VERSION=0

function usage() {
	echo "
usage: ./$0 [-b] [-r] [-u] [-c] [-v]
	-b build
	-r rpmbuild
	-u upload
	-c clean
	-v version
"
}

function parse_args() {
	while getopts 'brucv:' opt; do
		case $opt in
			b) BUILD=1 ;;
			r) RPMBUILD=1 ;;
			u) UPLOAD=1 ;;
			c) CLEAN=1 ;;
			v) VERSION=$OPTARG ;;
			\?) usage && exit 0;;
		esac
	done
}

function main() {
	parse_args $@
	echo "BUILD=$BUILD"
	echo "RPMBUILD=$RPMBUILD"
	echo "UPLOAD=$UPLOAD"
	echo "CLEAN=$CLEAN"
	echo "VERSION=$VERSION"
}

main $@
```

以上脚本的执行结果如下：

```bash
# ./test.sh -v 1.1.1 -b
BUILD=1
RPMBUILD=0
UPLOAD=0
CLEAN=0
VERSION=1.1.1
```

不难看出，-b/-r/-u/-c只能作为开关变量，而-v后可以跟自定义参数，这是因为getopts的第一个参数为'brucv:'，v后面追加了一个":"。

getopt一般用于工程的入口脚本，用于构建一种产品各种形态的发布件，比如develop/release版本。

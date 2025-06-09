# 工程目录

```bash
# tree .
.
├── demo.c         # 源文件，编译生成 ./demo 可执行文件
├── demo.data      # 数据文件
├── demo.h         # 头文件
├── demo.man.1     # man手册
└── meson.build    # meson配置
```

# meson.build内容

```bash
# 工程名称为DEMO
project('DEMO', 'C')

# 编译demo.c生成的可执行文件为demo，安装在bin/目录下
executable('demo', 'demo.c', install : true, install_dir : 'bin')

# 将demo.h安装在include/目录下
install_headers('demo.h')

# 将demo.man.1安装在share/man/man1/目录下，要安装的文件必须以".X"结尾，X为数字，代表manX/目录，这里为man1/目录
install_man('demo.man.1')

# 将sbin子目录安装在"."目录下
install_subdir('sbin', install_dir : '.')

# 将demo.dat安装在data/目录下
install_data('demo.data', install_dir : 'data')
```

# meson构建和安装

- 指定构建目录为build/，`--prefix`指定安装目录的前缀，`--buildtype`指定构建的版本类型

```bash
meson setup build --prefix=`pwd`/install --buildtype=debug
```

- 编译源码，-C 指定构建目录

```bash
meson compile -C build
```

- 安装软件，-C 指定构建目录

```bash
meson install -C build
```

- 构建完成，会在当前目录下生成build/目录和install/目录，build/目录存放构建的中间过程，install/目录存放安装的具体文件：

```bash
# tree install/
install/
├── bin
│   └── demo
├── data
│   └── demo.data
├── include
│   └── demo.h
├── sbin
└── share
    └── man
        └── man1
            └── demo.man.1
```

# meson其它命令

- 查看meson的默认配置，可以使用grep过滤选项

```bash
meson configure
meson configure | grep buildtype
```

也可参见：
https://mesonbuild.com/Installing.html#custom-install-behaviour
